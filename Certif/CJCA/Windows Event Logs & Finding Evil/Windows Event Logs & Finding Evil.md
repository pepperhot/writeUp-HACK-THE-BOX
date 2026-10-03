# 🎯 Fiche de révision HTB CJCA — Windows Event Logs & Finding Evil

> **Légende**
> - 📘 Résumé du cours (traduit en français)
> - 🔑 À retenir absolument
> - ⌨️ Requêtes / commandes importantes
> - ✅ Question, réponse en français, et **Réponse à mettre sur HTB** (en anglais, telle qu'attendue)
> - 🧭 Procédure complète, étape par étape, pour retrouver la réponse
> - ⚠️ Passage reconstitué (absent du cours ou réponse non communiquée). À vérifier sur la cible.

---

## Sommaire

1. [Page 1 — Windows Event Logs](#page-1--windows-event-logs)
2. [Page 2 — Analyzing Evil With Sysmon & Event Logs](#page-2--analyzing-evil-with-sysmon--event-logs)
3. [Page 3 — Event Tracing for Windows (ETW)](#page-3--event-tracing-for-windows-etw)
4. [Page 4 — Tapping Into ETW](#page-4--tapping-into-etw)
5. [Page 5 — Get-WinEvent](#page-5--get-winevent)
6. [Page 6 — Skills Assessment](#page-6--skills-assessment)
7. [Mémo final : toutes les réponses HTB](#mémo-final--toutes-les-réponses-htb)
8. [Cheat-sheet du module](#cheat-sheet-du-module)

---

# 🔌 Bloc de connexion à la cible (valable pour toutes les pages de ce module)

Toutes les questions pratiques de ce module se font en **RDP** sur une machine Windows cible.

**Identifiants fournis par le cours** (toujours les mêmes) :
- Utilisateur : `Administrator`
- Mot de passe : `HTB_@cad3my_lab_W1n10_r00t!@0`

## Option A — Depuis le Pwnbox (le plus simple)

| Étape | Action |
|---|---|
| A1 | En bas de la section, clique sur **Spawn Target** et note l'**IP cible** (ex. `10.129.xx.xx`) |
| A2 | Clique sur **Linux Pwnbox** (ou **Windows Pwnbox**) → **View** et attends le chargement |
| A3 | Ouvre un terminal dans le Pwnbox |
| A4 | Connecte-toi en RDP : |

```bash
xfreerdp /u:Administrator /p:'HTB_@cad3my_lab_W1n10_r00t!@0' /v:IP_CIBLE /dynamic-resolution
```

Test de connectivité avant le RDP si besoin :
```bash
ping -c 2 IP_CIBLE
nmap -Pn -p 3389 IP_CIBLE
# Attendu : 3389/tcp open ms-wbt-server
```

## Option B — Depuis ta propre machine (VPN)

| Étape | Commande / action |
|---|---|
| B1 | Section du cours → bouton **OVPN** → **View VPN** → **Download VPN Connection File** |
| B2 | Dans un terminal (le laisser ouvert) : |

```bash
sudo openvpn ~/Downloads/NOM_DU_FICHIER.ovpn
# Attendu à la fin : "Initialization Sequence Completed"
```

| Étape | Commande / action |
|---|---|
| B3 | Dans un second terminal, vérifie l'interface VPN : |

```bash
ip -4 addr show tun0
# Attendu : une adresse du type 10.10.14.x
```

| Étape | Commande / action |
|---|---|
| B4 | **Spawn Target**, note l'IP cible, teste la connexion : |

```bash
ping -c 2 IP_CIBLE
nmap -Pn -p 3389 IP_CIBLE
```

| Étape | Commande / action |
|---|---|
| B5 | Connexion RDP : |

```bash
xfreerdp /u:Administrator /p:'HTB_@cad3my_lab_W1n10_r00t!@0' /v:IP_CIBLE /dynamic-resolution
```

✔️ **Plan B si `xfreerdp` échoue** (erreur de certificat, de sécurité, ou de résolution) :
```bash
xfreerdp /u:Administrator /p:'HTB_@cad3my_lab_W1n10_r00t!@0' /v:IP_CIBLE /dynamic-resolution /cert:ignore
```
ou, si la négociation de sécurité échoue :
```bash
xfreerdp /u:Administrator /p:'HTB_@cad3my_lab_W1n10_r00t!@0' /v:IP_CIBLE /dynamic-resolution /sec:nla /cert:ignore
```

## Ouvrir l'Event Viewer une fois connecté en RDP

| Étape | Action |
|---|---|
| E1 | Clique sur la loupe de recherche Windows (en bas à gauche) |
| E2 | Tape **Event Viewer** |
| E3 | Clic droit → **Run as administrator** (ou clic gauche si déjà admin) |
| E4 | Dans le volet gauche, déplie **Windows Logs** pour Application / Security / Setup / System, ou **Applications and Services Logs → Microsoft → Windows → Sysmon → Operational** pour les logs Sysmon |

---

# Page 1 — Windows Event Logs

## 📘 Résumé

### Les bases du Windows Event Logging
- Les **Windows Event Logs** font partie intégrante du système d'exploitation. Ils stockent les logs du système lui-même, des applications, des providers **ETW**, des services, etc.
- Ils couvrent les erreurs applicatives, les événements de sécurité et les informations de diagnostic.
- Accès : application **Event Viewer** (interface graphique) ou **API Windows Event Log** (programmatique).

### Les 5 logs par défaut
| Log | Contenu |
|---|---|
| **Application** | Erreurs et informations des applications |
| **Security** | Événements de sécurité (logons, audits…) |
| **Setup** | Activités d'installation/configuration du système |
| **System** | Informations générales sur le système |
| **Forwarded Events** | Logs **transférés depuis d'autres machines** (vue centralisée pour les admins) |

🔑 L'Event Viewer peut aussi **ouvrir des fichiers `.evtx`** déjà sauvegardés, dans la section **Saved Logs**.

### Les 2 niveaux d'événements dans les logs d'application
| Niveau | Description |
|---|---|
| **Information** | Détails d'usage général (démarrage/arrêt de l'application) |
| **Error** | Erreurs spécifiques avec détails techniques |

### Anatomie d'un événement (les champs principaux)
| Champ | Description |
|---|---|
| **Log Name** | Nom du log (Application, System, Security…) |
| **Source** | Logiciel qui a généré l'événement |
| **Event ID** | Identifiant unique de l'événement |
| **Task Category** | Aide à comprendre le but/l'usage de l'événement |
| **Level** | Sévérité (Information, Warning, Error, Critical, Verbose) |
| **Keywords** | Étiquettes de catégorisation (ex. « Audit Success », « Audit Failure » dans Security) |
| **User** | Compte connecté au moment de l'événement |
| **OpCode** | Opération spécifique rapportée par l'événement |
| **Logged** | Date et heure de l'enregistrement |
| **Computer** | Nom de la machine |
| **XML Data** | Toutes ces infos + données supplémentaires, en XML |

🔑 Le champ **Keywords** est particulièrement utile pour **filtrer** les logs avec précision.

### Exemple d'analyse : Event ID 4624 (Successful Logon)
- Signifie la **création d'une session de logon** sur la machine de destination.
- Champs clés :
  - **Logon ID** : permet de **corréler** cet événement avec d'autres événements partageant le même Logon ID.
  - **Logon Type** : type de logon (ex. **Type 5 = Service logon**, initié par SYSTEM pour démarrer un nouveau service).
- Pour savoir **quel service** précisément, il faut corréler avec d'autres événements via le Logon ID.

### Les requêtes XML personnalisées
Chemin dans l'Event Viewer : **Filter Current Log** → **XML** → **Edit Query Manually**.

**Exemple du cours** : filtrer sur `SubjectLogonId = 0x3E7` (le Logon ID du compte SYSTEM) pour suivre toutes les actions liées à cette session.

🔑 On peut aussi cocher des filtres automatiques dans l'interface pour voir comment ils se traduisent en XML, et s'en inspirer.

### Fil rouge de l'exemple du cours (Event ID 4624 → 4907 → 4624/4672)
| Étape | Event ID | Ce qu'on apprend |
|---|---|---|
| 1 | **4624** | Logon Type 5 (Service), compte SYSTEM. Logon ID à noter : `0x3E7` |
| 2 | Filtre XML sur `SubjectLogonId = 0x3E7` | Isole tous les événements liés à cette session |
| 3 | **4907** (Audit Policy Change) | Le **SACL** (System Access Control List) d'un objet (fichier/clé de registre) a été modifié. Processus responsable : **SetupHost.exe**. Objet modifié : le **bootmanager**. Champs `NewSd` / `OldSd` = nouveau/ancien descripteur de sécurité |
| 4 | **4624** puis **4672** (Special Logon) | Un logon suivi d'un logon spécial, indiquant des privilèges élevés attribués |
| 5 | Détail de 4672 | Liste de privilèges accordés, ex. **SeDebugPrivilege** = capacité de manipuler la mémoire d'autres processus |

🔑 **SACL** : liste de contrôle d'accès **système**, qui permet de journaliser les tentatives d'accès à un objet sécurisé (succès, échec, ou les deux).
🔑 **Attention** : un nom de processus légitime (ex. `SetupHost.exe`) peut être usurpé par un malware (masquerade).

### Liste indicative des Event IDs utiles

**Windows System Logs**
| Event ID | Nom | Utilité défensive |
|---|---|---|
| **1074** | System Shutdown/Restart | Détecter des arrêts/redémarrages inattendus |
| **6005** | Event log service started | Marque un démarrage système, point de départ d'investigation |
| **6006** | Event log service stopped | Normal à l'extinction ; anormal sinon → possible dissimulation |
| **6013** | Windows uptime (quotidien) | Une durée de fonctionnement plus courte que prévu = reboot suspect |
| **7040** | Service status change | Changement manuel/automatique = signe de manipulation |

**Windows Security Logs**
| Event ID | Nom | Utilité défensive |
|---|---|---|
| **1102** | Audit log cleared | Tentative d'effacer les preuves |
| **1116 / 1118 / 1119 / 1120** | Defender : détection / début remédiation / succès / échec remédiation | Suivre le cycle de vie d'une détection antivirus |
| **4624 / 4625** | Logon réussi / échoué | Comportement normal vs brute force |
| **4648** | Logon avec identifiants explicites | Indice de mouvement latéral |
| **4656** | Handle demandé sur un objet | Accès à des ressources sensibles |
| **4672** | Privilèges spéciaux attribués | Suivi des comptes à privilèges élevés |
| **4698 / 4700 / 4701 / 4702** | Tâche planifiée créée / activée / désactivée / mise à jour | Mécanisme de **persistance** classique |
| **4719** | Politique d'audit modifiée | Tentative de dissimulation |
| **4738** | Compte utilisateur modifié | Prise de contrôle de compte / menace interne |
| **4771** | Pré-authentification Kerberos échouée | Brute force Kerberos |
| **4776** | Validation d'identifiants par le DC | Brute force sur le contrôleur de domaine |
| **5001** | Config protection temps réel Defender modifiée | Tentative de désactivation de l'AV |
| **5140 / 5142 / 5145** | Accès / création / vérification de partage réseau | Exfiltration, reconnaissance de partages |
| **5157** | Connexion bloquée par le Windows Filtering Platform | Trafic malveillant bloqué |
| **7045** | Service installé | Installation de malware en tant que service |

🔑 **Règle d'or** : bien connaître ce qui est **normal** dans son environnement pour repérer les anomalies et réduire les faux positifs. Centraliser les logs, les corréler, et surveiller en continu.

## ✅ Questions de la page 1

### ❓ Question 1 — L'exécutable responsable de la modification des paramètres d'audit

| | |
|---|---|
| **Question (EN)** | Analyze the event with ID 4624, that took place on 8/3/2022 at 10:23:25. Conduct a similar investigation as outlined in this section and provide the name of the executable responsible for the modification of the auditing settings as your answer. Answer format: T_W_____.exe |
| **Question (FR)** | Analyse l'événement d'ID 4624 du 8/3/2022 à 10:23:25. Mène une investigation similaire à celle du cours et donne le nom de l'exécutable responsable de la modification des paramètres d'audit. |
| **Réponse (FR)** | `TiWorker.exe` |
| **Réponse à mettre sur HTB** | `TiWorker.exe` |

**🧭 Procédure complète**

**Partie 0 : connexion à la cible**

1. **Spawn Target** → note `IP_CIBLE`.
2. Connexion RDP :
   ```bash
   xfreerdp /u:Administrator /p:'HTB_@cad3my_lab_W1n10_r00t!@0' /v:IP_CIBLE /dynamic-resolution
   ```

**Partie 1 : retrouver l'événement 4624 du 8/3/2022 à 10:23:25**

3. Loupe Windows → **Event Viewer** → Exécuter en administrateur.
4. Volet gauche → **Windows Logs** → **Security**.
5. Clic droit sur **Security** → **Filter Current Log...**.
6. Dans **Logged**, choisis **Custom Range** et saisis la date **8/3/2022** autour de **10:23:25** (quelques minutes avant/après pour être sûr de le voir).
7. Dans **\<All Event IDs\>**, saisis **4624** → **OK**.
8. Repère dans la liste la ligne avec l'heure exacte **10:23:25** et double-clique dessus pour l'ouvrir.
9. Note le champ **Logon ID** de cet événement (ex. `0x3E7` ou une autre valeur hexadécimale propre à cette session).

**Partie 2 : isoler la session avec une requête XML**

10. Reviens sur **Security** → clic droit → **Filter Current Log...** → onglet **XML** → coche **Edit query manually** → **Yes**.
11. Remplace la requête par (adapte `VALEUR_LOGON_ID` au Logon ID trouvé à l'étape 9) :
    ```xml
    <QueryList>
      <Query Id="0" Path="Security">
        <Select Path="Security">*[EventData[Data[@Name='SubjectLogonId']='VALEUR_LOGON_ID']]</Select>
      </Query>
    </QueryList>
    ```
12. **OK**. La liste se réduit à tous les événements liés à cette session.

**Partie 3 : suivre le fil rouge jusqu'à l'Event ID 4907**

13. Parcours les résultats chronologiquement (trie par colonne **Date and Time**).
14. Repère un événement **4907** (Audit Policy Change) : « This event generates when the SACL of an object… was changed ».
15. Ouvre le détail de cet événement et lis le champ **Process Name** : c'est l'exécutable responsable de la modification de l'audit.
16. Vérifie que le résultat correspond au format demandé `T_W_____.exe` → `TiWorker.exe` (TiWorker = Windows Modules Installer Worker, processus légitime de Windows Update).

**Partie 4 : validation**

17. Saisis `TiWorker.exe` sur HTB → **Submit**.

✔️ **Plan B** si le filtre XML ne retourne rien : vérifie que `SubjectLogonId` est bien le nom du champ (parfois `TargetLogonId` selon l'événement) en inspectant l'onglet **Details → XML View** d'un événement 4624 ouvert.

---

### ❓ Question 2 — L'heure de la modification de l'audit sur un fichier précis

| | |
|---|---|
| **Question (EN)** | Build an XML query to determine if the previously mentioned executable modified the auditing settings of C:\Windows\Microsoft.NET\Framework64\v4.0.30319\WPF\wpfgfx_v0400.dll. Enter the time of the identified event in the format HH:MM:SS as your answer. |
| **Question (FR)** | Construis une requête XML pour déterminer si l'exécutable précédent a modifié les paramètres d'audit de `C:\Windows\Microsoft.NET\Framework64\v4.0.30319\WPF\wpfgfx_v0400.dll`. Donne l'heure de l'événement trouvé (HH:MM:SS). |
| **Réponse (FR)** | `10:23:50` |
| **Réponse à mettre sur HTB** | `10:23:50` |

**🧭 Procédure complète**

**Partie 0 : connexion** (identique à la question 1, si pas déjà connecté)

```bash
xfreerdp /u:Administrator /p:'HTB_@cad3my_lab_W1n10_r00t!@0' /v:IP_CIBLE /dynamic-resolution
```

**Partie 1 : construire la requête XML**

1. **Event Viewer** → **Security** → clic droit → **Filter Current Log...** → onglet **XML** → **Edit query manually**.
2. On cherche un événement **4907** (modification de SACL) qui porte sur le fichier `wpfgfx_v0400.dll` **et** dont le processus est `TiWorker.exe`. Requête :
   ```xml
   <QueryList>
     <Query Id="0" Path="Security">
       <Select Path="Security">
         *[System[(EventID=4907)]]
         and
         *[EventData[Data[@Name='ObjectName']='C:\Windows\Microsoft.NET\Framework64\v4.0.30319\WPF\wpfgfx_v0400.dll']]
       </Select>
     </Query>
   </QueryList>
   ```
3. **OK**.
4. Si plusieurs résultats apparaissent (plusieurs processus ont pu toucher ce fichier), ouvre chacun et vérifie le champ **Process Name** = `TiWorker.exe` pour confirmer que c'est bien le bon exécutable.
5. Repère l'événement correspondant et lis le champ **Date and Time** (ou **Logged**) : l'heure affichée est `10:23:50`.

**Partie 2 : validation**

6. Saisis `10:23:50` sur HTB → **Submit**.

✔️ **Plan B** : si le nom du champ `ObjectName` ne matche pas (orthographe du chemin, antislashs), ouvre un événement **4907** existant en vue **XML** (onglet Details → XML View) pour copier le nom exact du champ et la syntaxe du chemin telle qu'elle apparaît réellement dans les logs.

---

# Page 2 — Analyzing Evil With Sysmon & Event Logs

## 📘 Résumé

### Bases de Sysmon
- **Sysmon** (System Monitor) = service Windows + pilote qui reste actif après redémarrage, et journalise l'activité système dans l'Event Log Windows.
- Capture des infos qui **n'apparaissent pas dans les logs Security classiques** : création de processus, connexions réseau, changements de date de création de fichier, etc.
- 3 composants : le **service** Windows, le **pilote** (driver), et l'**event log** qui affiche les données.
- Chaque **Event ID** Sysmon correspond à un type d'activité précis (ex. **ID 1** = Process Creation, **ID 3** = Network Connection).

### Installation et configuration
```powershell
C:\Tools\Sysmon> sysmon.exe -i -accepteula -h md5,sha256,imphash -l -n
```
- `-i` : installation. `-h` : algorithmes de hash à calculer. `-l` : logger les chargements de modules (ImageLoad). `-n` : logger les connexions réseau.

Pour utiliser une configuration personnalisée (fichier XML) :
```powershell
C:\Tools\Sysmon> sysmon.exe -c filename.xml
```

🔑 Configs de référence :
- **SwiftOnSecurity** : https://github.com/SwiftOnSecurity/sysmon-config (utilisée dans le cours)
- **olafhartong/sysmon-modular** : approche modulaire

🔑 Sysmon existe aussi **pour Linux**.

### Détection 1 : DLL Hijacking (Event ID 7 — Image Loaded)

**Principe** : un attaquant place une DLL malveillante portant le **même nom** qu'une DLL légitime, dans un dossier où le programme la cherchera **avant** le dossier System32.

**Config Sysmon** : dans `sysmonconfig-export.xml`, il faut passer la règle `ImageLoad` de **include** à **exclude** (sans règle), pour **tout capturer** au lieu de ne rien capturer par défaut.
```powershell
C:\Tools\Sysmon> sysmon.exe -c sysmonconfig-export.xml
```
Les événements apparaissent dans : **Event Viewer → Applications and Services Logs → Microsoft → Windows → Sysmon → Operational**, Event ID **7**.

**Exemple du cours : `calc.exe` + `WININET.dll`**
1. Renommer `reflective_dll.x64.dll` en `WININET.dll`.
2. Copier `calc.exe` (normalement dans `System32`) et la DLL malveillante dans un dossier **inscriptible** (ex. Bureau).
3. Lancer `calc.exe` depuis ce dossier → au lieu de la calculatrice, une **MessageBox** apparaît (preuve que la DLL malveillante a été chargée).

**Les 3 IOCs identifiés**
| IOC | Détail |
|---|---|
| **Emplacement de calc.exe** | Ne devrait jamais se trouver ailleurs que dans `System32` (ou `Syswow64`) |
| **Emplacement de WININET.dll** | Chargée par `calc.exe` **hors de** `System32` = hijack confirmé (le nom de la DLL ne peut pas être changé par l'attaquant sans casser le hijack) |
| **Signature** | La DLL originale est **signée Microsoft** ; la DLL injectée est **non signée** |

**Filtrage dans l'Event Viewer**
1. **Filter Current Log...** → logs **Microsoft-Windows-Sysmon/Operational** → **Event ID 7**.
2. **Find...** → rechercher `calc.exe`.
3. Comparer un chargement **légitime** de `wininet.dll` (signé `true`, dans System32) à un chargement **malveillant** de `WININET.dll` (signé `false`, hors System32).

### Détection 2 : Injection PowerShell non managée / C#

- **C#** est un langage **managé** : il nécessite le **CLR** (Common Language Runtime) pour s'exécuter. Le code managé est compilé en bytecode, exécuté par le runtime.
- Outil utilisé : **Process Hacker**. Un processus managé (.NET) apparaît en **vert**, avec l'infobulle « Process is managed (.NET) ».
- Dans **Process Hacker → clic droit sur le processus → Properties → Modules**, repérer le chargement de **clr.dll** et **clrjit.dll** : ces DLLs ne devraient être chargées que dans des processus qui utilisent réellement .NET.
- Si on les trouve dans un processus qui n'en a normalement pas besoin (ex. `powershell.exe` classique est **managé** ; mais **`spoolsv.exe`** ne l'est normalement **pas**) → signe d'injection.

**Exemple du cours : injection dans `spoolsv.exe` avec PSInject**
```powershell
powershell -ep bypass
Import-Module .\Invoke-PSInject.ps1
Invoke-PSInject -ProcId [PID de spoolsv.exe] -PoshCode "V3JpdGUtSG9zdCAiSGVsbG8sIEd1cnU5OSEi"
```
- Après l'injection, `spoolsv.exe` passe de **non managé** à **managé** (visible dans Process Hacker ou via Sysmon Event ID 7 : chargement de `clr.dll`).

### Détection 3 : Credential Dumping (Mimikatz, Event ID 10 — ProcessAccess)

**Commande d'attaque**
```
C:\Tools\Mimikatz> mimikatz.exe
mimikatz # privilege::debug
mimikatz # sekurlsa::logonpasswords
```
- `sekurlsa::logonpasswords` extrait les mots de passe/hash en accédant à **LSASS** (Local Security Authority Subsystem Service), qui gère les identifiants utilisateurs.

**Détection : Sysmon Event ID 10 (ProcessAccess)**
- On surveille les accès **suspects** à `lsass.exe` :
  - **SourceImage** inhabituel (ex. un exécutable aléatoire lancé depuis **Downloads**)
  - **SourceUser ≠ TargetUser** (ex. source = `waldo`, cible = `SYSTEM`)
  - Le processus attaquant doit généralement demander **SeDebugPrivilege** → IOC supplémentaire

⚠️ Certains processus légitimes (AV, EDR, outils d'authentification) accèdent aussi à LSASS : il faut du contexte pour éviter les faux positifs.

## ✅ Questions de la page 2

### ❓ Question 1 — Hash SHA256 de la DLL malveillante (DLL Hijacking)

| | |
|---|---|
| **Question (EN)** | Replicate the DLL hijacking attack described in this section and provide the SHA256 hash of the malicious WININET.dll as your answer. "C:\Tools\Sysmon" and "C:\Tools\Reflective DLLInjection" on the spawned target contain everything you need. |
| **Question (FR)** | Reproduis l'attaque de DLL hijacking décrite dans cette section et donne le hash SHA256 de la `WININET.dll` malveillante. |
| **Réponse (FR)** | Hash SHA256 de la DLL malveillante |
| **Réponse à mettre sur HTB** | `51F2305DCF385056C68F7CCF5B1B3B9304865CEF1257947D4AD6EF5FAD2E3B13` |

**🧭 Procédure complète**

**Partie 0 : connexion**
```bash
xfreerdp /u:Administrator /p:'HTB_@cad3my_lab_W1n10_r00t!@0' /v:IP_CIBLE /dynamic-resolution
```

**Partie 1 : configurer Sysmon pour capturer les Image Load**

1. Ouvre une invite de commandes **en administrateur** (clic droit → *Run as administrator*).
2. Va dans le dossier Sysmon :
   ```cmd
   cd C:\Tools\Sysmon
   ```
3. Installe Sysmon (si ce n'est pas déjà fait) :
   ```cmd
   sysmon.exe -i -accepteula -h md5,sha256,imphash -l -n
   ```
4. Édite (ou utilise une copie déjà modifiée de) `sysmonconfig-export.xml` pour passer la règle `ImageLoad` de `include` à `exclude` (vide), afin de tout capturer. Si le fichier modifié est déjà fourni dans le dossier, passe directement à l'étape suivante.
5. Applique la configuration :
   ```cmd
   sysmon.exe -c sysmonconfig-export.xml
   ```

**Partie 2 : reproduire le hijack**

6. Va dans `C:\Tools\Reflective DLLInjection` (ou le nom exact du dossier fourni).
7. Copie `calc.exe` depuis `C:\Windows\System32\calc.exe` vers le Bureau :
   ```cmd
   copy C:\Windows\System32\calc.exe "C:\Users\Administrator\Desktop\calc.exe"
   ```
8. Copie/renomme la DLL réflective en `WININET.dll` à côté de `calc.exe` sur le Bureau :
   ```cmd
   copy "C:\Tools\Reflective DLLInjection\reflective_dll.x64.dll" "C:\Users\Administrator\Desktop\WININET.dll"
   ```
9. Lance le `calc.exe` du Bureau (double-clic, ou) :
   ```cmd
   C:\Users\Administrator\Desktop\calc.exe
   ```
10. Une **MessageBox** doit apparaître (« Hello from DllMain! » ou similaire) : le hijack a fonctionné.

**Partie 3 : récupérer le hash SHA256**

11. Dans PowerShell :
    ```powershell
    Get-FileHash "C:\Users\Administrator\Desktop\WININET.dll" -Algorithm SHA256
    ```
12. Copie la valeur du champ `Hash`.

✔️ **Plan B** (confirmation via Sysmon) : ouvre **Event Viewer → Applications and Services Logs → Microsoft → Windows → Sysmon → Operational**, filtre sur **Event ID 7**, cherche `calc.exe` (Find...), ouvre l'événement où `ImageLoaded` = `WININET.dll` **hors** System32 : le champ **Hashes** contient le SHA256.

**Partie 4 : validation**

13. Saisis `51F2305DCF385056C68F7CCF5B1B3B9304865CEF1257947D4AD6EF5FAD2E3B13` sur HTB → **Submit**.

---

### ❓ Question 2 — Hash SHA256 de clrjit.dll (injection PowerShell non managée)

| | |
|---|---|
| **Question (EN)** | Replicate the Unmanaged PowerShell attack described in this section and provide the SHA256 hash of clrjit.dll that spoolsv.exe will load as your answer. "C:\Tools\Sysmon" and "C:\Tools\PSInject" on the spawned target contain everything you need. |
| **Question (FR)** | Reproduis l'attaque PowerShell non managée et donne le hash SHA256 de `clrjit.dll` que `spoolsv.exe` va charger. |
| **Réponse (FR)** | Hash SHA256 de clrjit.dll |
| **Réponse à mettre sur HTB** | `8A3CD3CF2249E9971806B15C75A892E6A44CCA5FF5EA5CA89FDA951CD2C09AA9` |

**🧭 Procédure complète**

**Partie 0 : connexion** (si pas déjà fait)
```bash
xfreerdp /u:Administrator /p:'HTB_@cad3my_lab_W1n10_r00t!@0' /v:IP_CIBLE /dynamic-resolution
```

**Partie 1 : trouver le PID de spoolsv.exe**

1. Ouvre le **Gestionnaire des tâches** ou **Process Hacker** (`C:\Tools\...` si fourni, sinon `taskmgr`).
2. Repère le processus **spoolsv.exe** et note son **PID** (colonne PID, à activer via *Plus de détails* si besoin).
   - Alternative en PowerShell :
     ```powershell
     Get-Process spoolsv | Select-Object Id
     ```

**Partie 2 : réaliser l'injection**

3. Ouvre PowerShell :
   ```powershell
   powershell -ep bypass
   ```
4. Va dans le dossier de l'outil :
   ```powershell
   cd "C:\Tools\PSInject"
   ```
5. Importe le module :
   ```powershell
   Import-Module .\Invoke-PSInject.ps1
   ```
6. Injecte (remplace `PID_SPOOLSV` par le PID noté à l'étape 2) :
   ```powershell
   Invoke-PSInject -ProcId PID_SPOOLSV -PoshCode "V3JpdGUtSG9zdCAiSGVsbG8sIEd1cnU5OSEi"
   ```
7. Vérifie dans Process Hacker que `spoolsv.exe` est maintenant **managé** (.NET) : clic droit → *Properties* → *Modules* → chercher `clr.dll` et `clrjit.dll`.

**Partie 3 : récupérer le hash de clrjit.dll**

8. Dans Process Hacker (onglet **Modules** de `spoolsv.exe`), repère la ligne `clrjit.dll` et note son **chemin complet** (ex. `C:\Windows\Microsoft.NET\Framework64\v4.0.30319\clrjit.dll`).
9. En PowerShell :
   ```powershell
   Get-FileHash "C:\Windows\Microsoft.NET\Framework64\v4.0.30319\clrjit.dll" -Algorithm SHA256
   ```

✔️ **Plan B** : via Sysmon Event ID 7, filtre sur `Image` = `spoolsv.exe` et `ImageLoaded` = `clrjit.dll` : le champ **Hashes** de l'événement contient directement le SHA256.

**Partie 4 : validation**

10. Saisis `8A3CD3CF2249E9971806B15C75A892E6A44CCA5FF5EA5CA89FDA951CD2C09AA9` sur HTB → **Submit**.

---

### ❓ Question 3 — Hash NTLM de l'Administrateur (Credential Dumping)

| | |
|---|---|
| **Question (EN)** | Replicate the Credential Dumping attack described in this section and provide the NTLM hash of the Administrator user as your answer. "C:\Tools\Sysmon" and "C:\Tools\Mimikatz" on the spawned target contain everything you need. |
| **Question (FR)** | Reproduis l'attaque de credential dumping et donne le hash NTLM de l'utilisateur Administrator. |
| **Réponse (FR)** | Hash NTLM de l'Administrateur |
| **Réponse à mettre sur HTB** | `5e4ffd54b3849aa720ed39f50185e533` |

**🧭 Procédure complète**

**Partie 0 : connexion** (si pas déjà fait)
```bash
xfreerdp /u:Administrator /p:'HTB_@cad3my_lab_W1n10_r00t!@0' /v:IP_CIBLE /dynamic-resolution
```

**Partie 1 : lancer Mimikatz**

1. Ouvre une invite de commandes **en administrateur**.
2. Va dans le dossier de l'outil :
   ```cmd
   cd C:\Tools\Mimikatz
   ```
3. Lance Mimikatz :
   ```cmd
   mimikatz.exe
   ```
4. Dans la console Mimikatz, active le privilège de debug :
   ```
   privilege::debug
   ```
   (doit répondre `Privilege '20' OK`)
5. Lance le dump des identifiants en mémoire :
   ```
   sekurlsa::logonpasswords
   ```

**Partie 2 : lire le hash NTLM**

6. Dans la sortie, cherche le bloc `User Name : Administrator`.
7. Dans la sous-section `msv :`, lis la ligne **`* NTLM :`** → c'est le hash demandé.

**Partie 3 : vérifier côté Sysmon (optionnel, pour la détection)**

8. **Event Viewer → Sysmon → Operational**, filtre sur **Event ID 10** (ProcessAccess), cherche `TargetImage` = `lsass.exe` : le `SourceImage` est `mimikatz.exe`, ce qui confirme l'attaque.

**Partie 4 : validation**

9. Saisis `5e4ffd54b3849aa720ed39f50185e533` sur HTB → **Submit**.

---

# Page 3 — Event Tracing for Windows (ETW)

> Cette page ne contient **aucune question pratique**. C'est une page théorique, base de connaissance pour la page suivante (« Tapping Into ETW »).

## 📘 Résumé

### Qu'est-ce qu'ETW ?
- **ETW** (Event Tracing for Windows) = mécanisme de traçage **haute performance**, intégré au noyau Windows.
- Capture des événements générés par des applications **user-mode** et des pilotes **kernel-mode**.
- Couvre bien plus que les logs classiques : appels système, création/fin de processus, activité réseau, modifications de fichiers/registre, etc.
- Impact minime sur les performances, adapté à la surveillance en temps réel.
- Outils d'exploitation : **Get-WinEvent** (PowerShell), Message Analyzer de Microsoft.

### Architecture et composants

```
Providers (A, B, C) → Sessions ETW ← Controllers
                           ↓
                      Trace Files (.ETL)
                           ↓
                       Consumer
```

| Composant | Rôle |
|---|---|
| **Controllers** | Pilotent les sessions ETW (démarrage/arrêt, activation des providers). Exemple : l'utilitaire **`logman.exe`** |
| **Providers** | Génèrent les événements et les écrivent dans les sessions. 4 types : |
| ↳ **MOF Providers** | Basés sur le format **MOF** (Managed Object Format) |
| ↳ **WPP Providers** | « Windows Software Trace Preprocessor », macros dans le code source, pour le traçage kernel-mode bas niveau |
| ↳ **Manifest-based Providers** | Basés sur des fichiers **manifestes XML** (approche moderne, flexible) |
| ↳ **TraceLogging Providers** | API simplifiée, overhead de code minimal |
| **Consumers** | S'abonnent à des événements précis pour les traiter/analyser. Par défaut → fichier **.ETL** |
| **Channels** | Conteneurs logiques organisant/filtrant les événements. Les consommateurs s'abonnent à des canaux spécifiques |
| **ETL files** | Fichiers de stockage durable des événements, pour analyse hors-ligne et investigation forensique |

🔑 **À retenir** :
- ETW fonctionne en **kernel-mode et user-mode**.
- Certains providers (très verbeux) sont **désactivés par défaut** pour ne pas surcharger le système.
- ETW peut être étendu avec des **providers personnalisés**.
- Seuls les événements d'un provider ETW dotés d'une propriété **Channel** peuvent être consommés par l'event log.

### Interagir avec ETW via `logman.exe`

**Lister les sessions de traçage actives** (le paramètre `-ets` est indispensable) :
```cmd
C:\Tools> logman.exe query -ets
```
→ Montre les sessions de type *Trace* en cours (Sysmon y apparaît, ex. `EventLog-Microsoft-Windows-Sysmon-Operational`, `SYSMON TRACE`, `SysmonDnsEtwSession`).

**Examiner une session précise** (ex. `EventLog-System`) :
```cmd
C:\Tools> logman.exe query "EventLog-System" -ets
```
→ Affiche les détails : **Name**, **Max Log Size**, **Log Location**, et les **providers souscrits**, avec pour chacun :
- **Name / Provider GUID** : identifiant unique du provider
- **Level** : niveau filtré (warning, informational, critical…)
- **KeywordsAny** : filtre par type d'événement généré

**Lister tous les providers disponibles** (plus de 1000 sous Windows 10) :
```cmd
C:\Tools> logman.exe query providers
```

**Filtrer par mot-clé avec `findstr`** :
```cmd
C:\Tools> logman.exe query providers | findstr "Winlogon"
```

**Examiner un provider précis** (Keywords disponibles, niveaux, PID qui l'utilisent) :
```cmd
C:\Tools> logman.exe query providers Microsoft-Windows-Winlogon
```

🔑 Alternative GUI : **Performance Monitor** (voir les sessions de traçage en cours, double-clic pour le détail, création de sessions « User Defined »). Alternative pour explorer les métadonnées de providers : **EtwExplorer**.

### Providers utiles pour la détection
| Provider | Utilité |
|---|---|
| **Microsoft-Windows-Kernel-Process** | Détection de process injection, process hollowing |
| **Microsoft-Windows-Kernel-File** | Accès fichiers non autorisé, ransomware, exfiltration |
| **Microsoft-Windows-Kernel-Network** | Exfiltration, connexions non autorisées, C2 |
| **Microsoft-Windows-SMBClient/SMBServer** | Mouvement latéral, exfiltration via partages |
| **Microsoft-Windows-DotNETRuntime** | Anomalies d'exécution .NET, assemblies malveillants |
| **OpenSSH** | Tentatives de connexion SSH, brute force |
| **Microsoft-Windows-VPN-Client** | Connexions VPN suspectes |
| **Microsoft-Windows-PowerShell** | Usage suspect de PowerShell, script block logging |
| **Microsoft-Windows-Kernel-Registry** | Persistance, modifications de clés de registre |
| **Microsoft-Windows-CodeIntegrity** | Chargement de pilotes/code non signé |
| **Microsoft-Antimalware-Service** | Problèmes/désactivation du service antimalware |
| **WinRM** | Mouvement latéral, exécution de commandes à distance |
| **Microsoft-Windows-TerminalServices-LocalSessionManager** | Activité RDP suspecte |
| **Microsoft-Windows-Security-Mitigations** | Tentatives de contournement des mitigations de sécurité |
| **Microsoft-Windows-DNS-Client** | DNS tunneling, requêtes DNS inhabituelles (C2) |

### Les providers « restreints »
- **Microsoft-Windows-Threat-Intelligence** : provider à très haute valeur, réservé aux processus avec le droit **PPL** (Protected Process Light).
- Devenir PPL est un processus lourd (validation Microsoft, signature Authenticode spéciale, pilote **ELAM**), réservé normalement aux éditeurs antimalware. Des contournements existent.
- Intérêt : données très granulaires sur les menaces, utile en DFIR, détecte des attaques qui échappent aux autres défenses.

---

# Page 4 — Tapping Into ETW

## 📘 Résumé

### Détection 1 : relations parent-enfant anormales

- Dans un environnement Windows standard, certains processus **ne lancent jamais** certains autres (ex. `calc.exe` ne lance normalement jamais `cmd.exe`).
- Référence utile : la mind map de Samir Bousseaden sur les relations parent-enfant courantes.
- Outil : **Process Hacker**, vue hiérarchique des processus.
- Exemple anormal : `spoolsv.exe` qui crée `whoami.exe` au lieu de son `conhost.exe` habituel.

**Technique démontrée : Parent PID Spoofing** (via le projet `psgetsystem`)
```powershell
PS C:\Tools\psgetsystem> powershell -ep bypass
PS C:\Tools\psgetsystem> Import-Module .\psgetsys.ps1
PS C:\Tools\psgetsystem> [MyProcess]::CreateProcessFromParent([PID de spoolsv.exe],"C:\Windows\System32\cmd.exe","")
```
- **Sysmon Event 1** se fait tromper : il affiche `spoolsv.exe` comme parent de `cmd.exe`, alors qu'en réalité c'est **`powershell.exe`** qui l'a créé.

**Solution : ETW via SilkETW, provider `Microsoft-Windows-Kernel-Process`**
```cmd
c:\Tools\SilkETW_SilkService_v8\v8\SilkETW>SilkETW.exe -t user -pn Microsoft-Windows-Kernel-Process -ot file -p C:\windows\temp\etw.json
```
- Le fichier `etw.json` généré révèle la **vraie** relation : `powershell.exe` est bien le créateur de `cmd.exe`, contrairement à ce qu'affichait Sysmon.
- 🔑 **À retenir** : ETW peut révéler des informations plus exactes que Sysmon lorsque des techniques d'évasion (comme le spoofing de PPID) sont utilisées.
- On peut trouver le nom exact d'un provider avec : `logman.exe query providers | findstr "Process"`.
- Les logs SilkETW peuvent être ingérés par l'Event Viewer via **SilkService**.

### Détection 2 : chargement malveillant d'assembly .NET

**Contexte : Living off the Land (LotL) vs Bring Your Own Land (BYOL)**
| Approche | Principe |
|---|---|
| **LotL** | Utiliser des outils **déjà présents** sur le système (ex. PowerShell) pour réduire les soupçons |
| **BYOL** (terme de Mandiant) | Utiliser des **assemblies .NET** personnalisées, exécutées **entièrement en mémoire**, indépendantes des outils déjà en place |

**Pourquoi BYOL fonctionne bien**
- .NET est **préinstallé** sur chaque Windows.
- Le CLR gère la mémoire (garbage collection), pas besoin de gérer ça manuellement.
- Les assemblies .NET peuvent être chargées **directement en mémoire**, sans écriture sur disque → minimise les artefacts.
- .NET fournit des bibliothèques riches (HTTP, crypto, IPC) qui facilitent la création d'outils d'attaque sophistiqués.

**Exemple emblématique** : la commande **`execute-assembly`** de **Cobalt Strike**, qui exécute des assemblies .NET directement depuis la mémoire.

**Détection via Sysmon Event ID 7 (Image Loaded)**
- Surveiller le chargement de **`clr.dll`** et **`mscoree.dll`** (DLLs liées au runtime .NET) dans des processus qui n'en ont normalement pas besoin.

**Démonstration : Seatbelt (outil .NET de reconnaissance)**
```powershell
PS C:\Tools\GhostPack Compiled Binaries>.\Seatbelt.exe TokenPrivileges
```
- Déclenche le chargement de `clr.dll` et `mscoree.dll`, visible en Sysmon Event ID 7.

**Limite de Sysmon** : il indique **qu'une** DLL a été chargée, mais pas le **contenu** de l'assembly exécutée (quelles méthodes, quelles classes…), et génère un volume important d'événements.

**Solution : ETW via SilkETW, provider `Microsoft-Windows-DotNETRuntime`**
```cmd
c:\Tools\SilkETW_SilkService_v8\v8\SilkETW>SilkETW.exe -t user -pn Microsoft-Windows-DotNETRuntime -uk 0x2038 -ot file -p C:\windows\temp\etw.json
```
- `-uk 0x2038` = masque de **keywords** sélectionné, qui active 4 catégories :

| Keyword | Rôle |
|---|---|
| **JitKeyword** | Événements de compilation **JIT** (Just-In-Time) : quelles méthodes sont compilées à l'exécution |
| **InteropKeyword** | Interopérabilité code managé ↔ non managé (appels API natifs) |
| **LoaderKeyword** | Détails du **chargement des assemblies** par le runtime .NET |
| **NGenKeyword** | Événements liés aux assemblies **précompilées** (Native Image Generator), utile pour détecter l'évasion des détections basées sur JIT |

- Le fichier `etw.json` généré contient des **noms de méthodes** (ex. `ManagedInteropMethodName`), bien plus granulaire que Sysmon.

## ✅ Question de la page 4

### ❓ Question 1 — Nom de méthode ManagedInteropMethodName

| | |
|---|---|
| **Question (EN)** | Replicate executing Seatbelt and SilkETW as described in this section and provide the ManagedInteropMethodName that starts with "G" and ends with "ion" as your answer. "c:\Tools\SilkETW_SilkService_v8\v8" and "C:\Tools\GhostPack Compiled Binaries" on the spawned target contain everything you need. |
| **Question (FR)** | Reproduis l'exécution de Seatbelt et SilkETW décrite dans cette section et donne le `ManagedInteropMethodName` qui commence par « G » et finit par « ion ». |
| **Réponse (FR)** | `GetTokenInformation` |
| **Réponse à mettre sur HTB** | `GetTokenInformation` |

**🧭 Procédure complète**

**Partie 0 : connexion**
```bash
xfreerdp /u:Administrator /p:'HTB_@cad3my_lab_W1n10_r00t!@0' /v:IP_CIBLE /dynamic-resolution
```

**Partie 1 : lancer la capture ETW**

1. Ouvre une invite de commandes **en administrateur**.
2. Va dans le dossier SilkETW :
   ```cmd
   cd "c:\Tools\SilkETW_SilkService_v8\v8\SilkETW"
   ```
3. Lance la capture sur le provider `Microsoft-Windows-DotNETRuntime` avec le masque de keywords du cours :
   ```cmd
   SilkETW.exe -t user -pn Microsoft-Windows-DotNETRuntime -uk 0x2038 -ot file -p C:\windows\temp\etw.json
   ```
4. **Laisse cette fenêtre ouverte** (la capture tourne en continu).

**Partie 2 : exécuter Seatbelt dans une autre fenêtre**

5. Ouvre une **seconde** invite de commandes (ou PowerShell), toujours en administrateur.
6. Va dans le dossier de l'outil :
   ```powershell
   cd "C:\Tools\GhostPack Compiled Binaries"
   ```
7. Lance Seatbelt avec le module `TokenPrivileges` (comme dans le cours) :
   ```powershell
   .\Seatbelt.exe TokenPrivileges
   ```
8. Laisse l'exécution se terminer, puis retourne dans la fenêtre SilkETW et arrête la capture (**Ctrl+C**).

**Partie 3 : analyser le fichier JSON généré**

9. Ouvre le fichier de sortie :
   ```powershell
   notepad C:\windows\temp\etw.json
   ```
   ou, pour chercher directement en PowerShell :
   ```powershell
   Select-String -Path C:\windows\temp\etw.json -Pattern "ManagedInteropMethodName"
   ```
10. Dans les résultats, cherche une valeur de `ManagedInteropMethodName` qui commence par **« G »** et se termine par **« ion »**.
11. La valeur trouvée est `GetTokenInformation` (fonction Windows liée à la récupération des informations de jeton/privilèges, cohérente avec le module `TokenPrivileges` de Seatbelt).

**Partie 4 : validation**

12. Saisis `GetTokenInformation` sur HTB → **Submit**.

✔️ **Plan B** : si `Select-String` ne trouve rien, le fichier JSON peut contenir plusieurs objets par ligne. Ouvre-le avec Notepad++ (si disponible) ou utilise :
```powershell
Get-Content C:\windows\temp\etw.json | Select-String "G.*ion" | Select-String "ManagedInteropMethodName"
```

---

# Page 5 — Get-WinEvent

## 📘 Résumé

### Pourquoi Get-WinEvent ?
- Les grandes organisations génèrent des **millions de logs par jour**. Il faut des outils pour analyser **en masse**.
- **`Get-WinEvent`** (cmdlet PowerShell) interroge les logs classiques (System, Application…), les logs générés par la technologie Windows Event Log, et les logs **ETW**.

### Lister les logs et les providers disponibles
```powershell
PS> Get-WinEvent -ListLog * | Select-Object LogName, RecordCount, IsClassicLog, IsEnabled, LogMode, LogType | Format-Table -AutoSize
```
→ Colonnes utiles : **LogName**, **RecordCount**, **IsClassicLog** (format `.evt` vs `.evtx`), **IsEnabled**, **LogMode** (Circular / Retain / AutoBackup), **LogType** (Administrative / Analytical / Debug / Operational).

```powershell
PS> Get-WinEvent -ListProvider * | Format-Table -AutoSize
```
→ Liste les providers et les logs auxquels ils sont associés (**LogLinks**).

### Récupérer des événements

**Depuis un log nommé, limité à N événements, avec colonnes choisies :**
```powershell
PS> Get-WinEvent -LogName 'System' -MaxEvents 50 | Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message | Format-Table -AutoSize
```

**Depuis un log ETW/Operational :**
```powershell
PS> Get-WinEvent -LogName 'Microsoft-Windows-WinRM/Operational' -MaxEvents 30 | Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message | Format-Table -AutoSize
```

**Les événements les plus anciens (`-Oldest`)** — évite de trier manuellement :
```powershell
PS> Get-WinEvent -LogName 'Microsoft-Windows-WinRM/Operational' -Oldest -MaxEvents 30 | Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message | Format-Table -AutoSize
```

**Depuis un fichier `.evtx` exporté :**
```powershell
PS> Get-WinEvent -Path 'C:\Tools\chainsaw\EVTX-ATTACK-SAMPLES\Execution\exec_sysmon_1_lolbin_pcalua.evtx' -MaxEvents 5 | Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message | Format-Table -AutoSize
```

### Filtrer avec `-FilterHashtable`

**Par log et par ID(s) :**
```powershell
PS> Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; ID=1,3} | Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message | Format-Table -AutoSize
```
🔑 Voir les Event IDs **1** (Process Create) et **3** (Network Connection) rapprochés dans le temps peut indiquer une communication avec un **C2**.

**Sur un fichier `.evtx` exporté :**
```powershell
PS> Get-WinEvent -FilterHashtable @{Path='C:\...\sysmon_mshta_sharpshooter_stageless_meterpreter.evtx'; ID=1,3} | Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message | Format-Table -AutoSize
```

**Sur une plage de dates** (la date de fin est **exclusive**, donc prendre le jour suivant) :
```powershell
PS> $startDate = (Get-Date -Year 2023 -Month 5 -Day 28).Date
PS> $endDate   = (Get-Date -Year 2023 -Month 6 -Day 3).Date
PS> Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; ID=1,3; StartTime=$startDate; EndTime=$endDate} | Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message | Format-Table -AutoSize
```

### Filtrer avec `-FilterHashtable` + parsing XML manuel

Pour extraire des **champs précis** d'un événement Sysmon (ex. IP source/destination d'une connexion réseau, Event ID 3) :
```powershell
PS> Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; ID=3} |
ForEach-Object {
$xml = [xml]$_.ToXml()
$eventData = $xml.Event.EventData.Data
New-Object PSObject -Property @{
    SourceIP = $eventData | Where-Object {$_.Name -eq "SourceIp"} | Select-Object -ExpandProperty '#text'
    DestinationIP = $eventData | Where-Object {$_.Name -eq "DestinationIp"} | Select-Object -ExpandProperty '#text'
    ProcessGuid = $eventData | Where-Object {$_.Name -eq "ProcessGuid"} | Select-Object -ExpandProperty '#text'
    ProcessId = $eventData | Where-Object {$_.Name -eq "ProcessId"} | Select-Object -ExpandProperty '#text'
}
}  | Where-Object {$_.DestinationIP -eq "52.113.194.132"}
```
🔑 Avec le **ProcessGuid** récupéré, on peut remonter à l'arbre de processus complet.
🔑 Référence pour les champs disponibles : la spécification du format **EVTX**.

### Filtrer avec `-FilterXml` (requête XML brute, via une chaîne `$Query`)
```powershell
PS> $Query = @"
    <QueryList>
        <Query Id="0">
            <Select Path="Microsoft-Windows-Sysmon/Operational">*[System[(EventID=7)]] and *[EventData[Data='mscoree.dll']] or *[EventData[Data='clr.dll']]
            </Select>
        </Query>
    </QueryList>
"@
PS> Get-WinEvent -FilterXml $Query | ForEach-Object {Write-Host $_.Message `n}
```
→ Reproduit en PowerShell la détection de chargement de `clr.dll`/`mscoree.dll` vue à la page « Tapping Into ETW ».

### Filtrer avec `-FilterXPath`

**Détection d'installation d'un outil Sysinternals (acceptation de l'EULA via `reg.exe`) :**
```powershell
PS> Get-WinEvent -LogName 'Microsoft-Windows-Sysmon/Operational' -FilterXPath "*[EventData[Data[@Name='Image']='C:\Windows\System32\reg.exe']] and *[EventData[Data[@Name='CommandLine']='`"C:\Windows\system32\reg.exe`" ADD HKCU\Software\Sysinternals /v EulaAccepted /t REG_DWORD /d 1 /f']]" | Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message | Format-Table -AutoSize
```

**Connexions réseau vers une IP précise (Sysmon Event ID 3) :**
```powershell
PS> Get-WinEvent -LogName 'Microsoft-Windows-Sysmon/Operational' -FilterXPath "*[System[EventID=3] and EventData[Data[@Name='DestinationIp']='52.113.194.132']]"
```

### Filtrer sur les valeurs de propriétés (`.Properties[N].Value`)

```powershell
PS> Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; ID=1} -MaxEvents 1 | Select-Object -Property *
```
→ Affiche **toutes** les propriétés disponibles d'un événement.

**Détecter du PowerShell encodé (`-enc`) dans la ligne de commande du parent :**
```powershell
PS> Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; ID=1} | Where-Object {$_.Properties[21].Value -like "*-enc*"} | Format-List
```
- `$_` = l'objet courant dans le pipeline.
- `.Properties[21].Value` = pour un événement Sysmon ID 1, l'index **21** correspond à **`ParentCommandLine`**.
- `-like "*-enc*"` = recherche par motif avec jokers `*`.
- `Format-List` = affichage en liste, plus lisible que `Format-Table` pour de longs champs.

🔑 **`-enc`** est un paramètre PowerShell courant pour fournir une **commande encodée en Base64**, souvent utilisé pour obfusquer du code malveillant.

## ✅ Question de la page 5

### ❓ Question 1 — Heure d'ajout du partage `\*\PRINT`

| | |
|---|---|
| **Question (EN)** | Utilize the Get-WinEvent cmdlet to traverse all event logs located within the "C:\Tools\chainsaw\EVTX-ATTACK-SAMPLES\Lateral Movement" directory and determine when the \*\PRINT share was added. Enter the time of the identified event in the format HH:MM:SS as your answer. |
| **Question (FR)** | Utilise `Get-WinEvent` pour parcourir tous les logs du dossier `C:\Tools\chainsaw\EVTX-ATTACK-SAMPLES\Lateral Movement` et détermine à quelle heure le partage `\*\PRINT` a été ajouté. |
| **Réponse (FR)** | `12:30:30` |
| **Réponse à mettre sur HTB** | `12:30:30` |

**🧭 Procédure complète**

**Partie 0 : connexion**
```bash
xfreerdp /u:Administrator /p:'HTB_@cad3my_lab_W1n10_r00t!@0' /v:IP_CIBLE /dynamic-resolution
```

**Partie 1 : identifier l'Event ID pertinent**

1. L'ajout d'un partage réseau correspond à l'**Event ID 5142** (« A network share object was added »), vu à la page 1 dans la liste des logs de sécurité utiles.

**Partie 2 : interroger tous les fichiers `.evtx` du dossier**

2. Ouvre PowerShell (pas besoin d'être forcément admin pour lire des fichiers `.evtx` locaux, mais utilise une session admin par simplicité).
3. Récupère la liste des fichiers du dossier et interroge chacun avec `-FilterHashtable` sur `Path` et `ID=5142` :
   ```powershell
   $files = Get-ChildItem "C:\Tools\chainsaw\EVTX-ATTACK-SAMPLES\Lateral Movement\*.evtx"
   foreach ($f in $files) {
     try {
       Get-WinEvent -FilterHashtable @{Path=$f.FullName; ID=5142} -ErrorAction Stop |
         Select-Object TimeCreated, ID, ProviderName, Message
     } catch {}
   }
   ```
   (Le `try/catch` évite que le script s'arrête si un fichier ne contient aucun événement 5142.)
4. Dans les résultats, repère l'événement dont le champ **Message** mentionne le partage **`\*\PRINT`** (ou `\\*\PRINT`, selon l'affichage).
5. Note le champ **TimeCreated** : l'heure affichée est `12:30:30`.

**Partie 3 : validation**

6. Saisis `12:30:30` sur HTB → **Submit**.

✔️ **Plan B** : si rien ne sort avec `ID=5142`, élargis la recherche à tous les event IDs liés aux partages réseau (**5140, 5142, 5145**) :
```powershell
foreach ($f in $files) {
  try {
    Get-WinEvent -FilterHashtable @{Path=$f.FullName; ID=5140,5142,5145} -ErrorAction Stop |
      Where-Object {$_.Message -like "*PRINT*"} |
      Select-Object TimeCreated, ID, Message
  } catch {}
}
```

---

# Page 6 — Skills Assessment

## 📘 Contexte

Le SOC manager demande d'analyser d'**anciens logs d'attaque** déjà présents sur la machine, dans `C:\Logs\*` (pas besoin de reproduire les attaques soi-même, contrairement aux pages précédentes). Chaque sous-dossier correspond à une attaque vue dans le module :

| Dossier | Attaque correspondante (page du cours) |
|---|---|
| `C:\Logs\DLLHijack` | DLL Hijacking (page 2) |
| `C:\Logs\PowershellExec` | Injection PowerShell non managée (page 2) |
| `C:\Logs\Dump` | Credential Dumping / LSASS (page 2) |
| `C:\Logs\StrangePPID` | Relation parent-enfant anormale (page 4) |

> ⚠️ **Important** : tu ne m'as pas communiqué les réponses validées pour ce Skills Assessment. Je ne les invente pas. Les cases **« Réponse à mettre sur HTB »** restent à `*(à compléter)*`. La méthode ci-dessous applique directement les techniques des pages 1, 2 et 4 à des logs déjà enregistrés (probablement des fichiers `.evtx` ou des logs déjà présents dans l'Event Viewer local). Envoie-moi tes résultats et je les intègre.

## ✅ Les 6 questions

### ❓ Question 1 — Processus responsable du DLL Hijacking

| | |
|---|---|
| **Question (EN)** | By examining the logs located in the "C:\Logs\DLLHijack" directory, determine the process responsible for executing a DLL hijacking attack. Enter the process name as your answer. Answer format: _.exe |
| **Question (FR)** | En examinant les logs du dossier `C:\Logs\DLLHijack`, détermine le processus responsable de l'attaque de DLL hijacking. |
| **Réponse à mettre sur HTB** | `*(à compléter)*` |

**🧭 Procédure complète**

**Partie 0 : connexion**
```bash
xfreerdp /u:Administrator /p:'HTB_@cad3my_lab_W1n10_r00t!@0' /v:IP_CIBLE /dynamic-resolution
```

**Partie 1 : localiser et ouvrir les logs**

1. Ouvre l'**Explorateur de fichiers** → va dans `C:\Logs\DLLHijack`.
2. Regarde les fichiers présents : s'il y a un ou plusieurs fichiers `.evtx`, note leur(s) nom(s) complet(s).

**Partie 2 : analyser avec Event Viewer (méthode graphique)**

3. Ouvre **Event Viewer** → clic droit sur **Saved Logs** (ou **Action → Open Saved Log...**) → sélectionne le(s) fichier(s) `.evtx` du dossier `DLLHijack`.
4. Applique la méthode de la page 2 (Détection 1) : **Filter Current Log...** → **Event ID 7** (Image Loaded).
5. Cherche les événements où une **DLL système connue** (ex. `WININET.dll`, ou toute DLL citée dans le tableau des DLLs détournables pour `calc.exe` : `CRYPTBASE.DLL`, `edputil.dll`, `MLANG.dll`, `PROPSYS.dll`, `Secur32.dll`, `SSPICLI.DLL`, `WININET.dll`) est chargée **depuis un dossier inhabituel** (pas `System32`) par un exécutable qui ne devrait pas s'y trouver non plus.
6. Le champ **Image** de cet événement donne le processus responsable du hijack.

**Partie 3 : analyse en PowerShell (méthode alternative, plus rapide)**

7. Ouvre PowerShell et liste tous les Event ID 7 du fichier :
   ```powershell
   Get-WinEvent -Path "C:\Logs\DLLHijack\NOM_DU_FICHIER.evtx" -FilterXPath "*[System[(EventID=7)]]" |
     Select-Object TimeCreated, Message | Format-List
   ```
8. Repère la ligne `Image:` (le `.exe`) associée à une `ImageLoaded:` suspecte, avec `Signed: false` et un chemin hors `System32`.

**Partie 4 : validation**

9. Le nom de l'exécutable (format `_.exe`) trouvé est ta réponse. Soumets-le sur HTB.

---

### ❓ Question 2 — Processus ayant exécuté du PowerShell non managé

| | |
|---|---|
| **Question (EN)** | By examining the logs located in the "C:\Logs\PowershellExec" directory, determine the process that executed unmanaged PowerShell code. Enter the process name as your answer. Answer format: _.exe |
| **Question (FR)** | En examinant les logs du dossier `C:\Logs\PowershellExec`, détermine le processus qui a exécuté du code PowerShell non managé. |
| **Réponse à mettre sur HTB** | `*(à compléter)*` |

**🧭 Procédure complète**

**Partie 0 : connexion** (si pas déjà fait)
```bash
xfreerdp /u:Administrator /p:'HTB_@cad3my_lab_W1n10_r00t!@0' /v:IP_CIBLE /dynamic-resolution
```

**Partie 1 : ouvrir les logs**

1. **Event Viewer → Open Saved Log...** → sélectionne le(s) `.evtx` dans `C:\Logs\PowershellExec`.

**Partie 2 : chercher les chargements de clr.dll / clrjit.dll**

2. Applique la méthode de la page 2 (Détection 2) : filtre **Event ID 7**, cherche les processus qui chargent **`clr.dll`** et **`clrjit.dll`** alors qu'ils n'en ont normalement pas besoin (comme `spoolsv.exe` dans l'exemple du cours — ici ce sera probablement un autre processus hôte).
3. En PowerShell, méthode rapide :
   ```powershell
   Get-WinEvent -Path "C:\Logs\PowershellExec\NOM_DU_FICHIER.evtx" -FilterXPath "*[System[(EventID=7)]] and *[EventData[Data='clr.dll']]" |
     Select-Object TimeCreated, Message | Format-List
   ```
4. Le processus qui a **chargé** `clr.dll`/`clrjit.dll` de façon inattendue (le processus « hôte » de l'injection, devenu managé) est celui qui a **exécuté** le code PowerShell non managé.

**Partie 3 : validation**

5. Le nom de l'exécutable trouvé (format `_.exe`) est ta réponse. Soumets-le sur HTB.

---

### ❓ Question 3 — Processus qui a injecté dans le processus précédent

| | |
|---|---|
| **Question (EN)** | By examining the logs located in the "C:\Logs\PowershellExec" directory, determine the process that injected into the process that executed unmanaged PowerShell code. Enter the process name as your answer. Answer format: _.exe |
| **Question (FR)** | Toujours dans `C:\Logs\PowershellExec`, détermine le processus qui a **injecté** dans le processus trouvé à la question précédente. |
| **Réponse à mettre sur HTB** | `*(à compléter)*` |

**🧭 Procédure complète**

**Partie 0 : connexion** (si pas déjà fait, identique aux questions précédentes)

**Partie 1 : chercher l'événement d'accès au processus cible**

1. Une injection de code dans un processus distant passe typiquement par un **accès au processus** (ouverture d'un handle avec des droits d'écriture/exécution) avant l'injection elle-même. Dans Sysmon, cela correspond à l'**Event ID 10** (ProcessAccess), comme vu pour LSASS à la page 2, mais ici appliqué au processus cible trouvé en Q2 (ex. `spoolsv.exe` ou équivalent).
2. Dans Event Viewer (ou via `Get-WinEvent`), filtre les événements **Event ID 10** où le champ **TargetImage** correspond au processus trouvé en Q2 :
   ```powershell
   Get-WinEvent -Path "C:\Logs\PowershellExec\NOM_DU_FICHIER.evtx" -FilterXPath "*[System[(EventID=10)]]" |
     Select-Object TimeCreated, Message | Format-List
   ```
3. Repère le champ **SourceImage** de l'événement dont le **TargetImage** est le processus de la question 2 : c'est le processus **injecteur**.

**Partie 2 : vérification croisée (Event ID 1, création de processus)**

4. Si l'Event ID 10 n'est pas présent dans ces logs (Sysmon ne capture pas toujours ProcessAccess selon la configuration), cherche plutôt l'**Event ID 1** (Process Creation) du processus trouvé en Q2 : le champ **ParentImage** peut indiquer l'injecteur, **sauf** s'il s'agit d'une injection dans un processus déjà existant (auquel cas Event ID 10 est la bonne piste, pas Event ID 1).

**Partie 3 : validation**

5. Le nom de l'exécutable trouvé (format `_.exe`) — probablement l'outil d'injection équivalent à `Invoke-PSInject.ps1` lancé depuis `powershell.exe` — est ta réponse. Soumets-le sur HTB.

---

### ❓ Question 4 — Processus ayant réalisé le dump LSASS

| | |
|---|---|
| **Question (EN)** | By examining the logs located in the "C:\Logs\Dump" directory, determine the process that performed an LSASS dump. Enter the process name as your answer. Answer format: _.exe |
| **Question (FR)** | En examinant les logs du dossier `C:\Logs\Dump`, détermine le processus qui a réalisé un dump de LSASS. |
| **Réponse à mettre sur HTB** | `*(à compléter)*` |

**🧭 Procédure complète**

**Partie 0 : connexion** (si pas déjà fait)

**Partie 1 : ouvrir les logs et filtrer sur ProcessAccess vers lsass.exe**

1. **Event Viewer → Open Saved Log...** → sélectionne le(s) `.evtx` dans `C:\Logs\Dump`.
2. Comme dans la page 2 (Détection 3), filtre sur **Event ID 10** (ProcessAccess) avec **TargetImage** = `lsass.exe` :
   ```powershell
   Get-WinEvent -Path "C:\Logs\Dump\NOM_DU_FICHIER.evtx" -FilterXPath "*[System[(EventID=10)]] and *[EventData[Data[@Name='TargetImage']='C:\Windows\system32\lsass.exe']]" |
     Select-Object TimeCreated, Message | Format-List
   ```
3. Lis le champ **SourceImage** : c'est le processus qui a accédé à LSASS pour en extraire les identifiants (l'outil de dump, ex. `mimikatz.exe` ou un équivalent renommé — vérifie bien le nom exact dans **ce** jeu de logs, il peut différer de l'exemple du cours).

✔️ **Plan B** si le champ `TargetImage` ne matche pas exactement (casse, chemin) : retire la condition sur `TargetImage` et filtre juste sur `EventID=10`, puis parcours manuellement les résultats pour repérer celui où `TargetImage` contient `lsass.exe`.

**Partie 2 : validation**

4. Le nom de l'exécutable trouvé (format `_.exe`) est ta réponse. Soumets-le sur HTB.

---

### ❓ Question 5 — Un login malveillant a-t-il eu lieu après le dump ?

| | |
|---|---|
| **Question (EN)** | By examining the logs located in the "C:\Logs\Dump" directory, determine if an ill-intended login took place after the LSASS dump. Answer format: Yes or No |
| **Question (FR)** | Toujours dans `C:\Logs\Dump`, détermine si un login malveillant a eu lieu **après** le dump de LSASS. |
| **Réponse à mettre sur HTB** | `*(à compléter — Yes ou No)*` |

**🧭 Procédure complète**

**Partie 0 : connexion** (si pas déjà fait)

**Partie 1 : noter l'heure du dump**

1. Reprends l'événement **Event ID 10** trouvé à la Question 4 (accès à LSASS) et note son **TimeCreated** précis.

**Partie 2 : chercher les logons après cette heure**

2. Filtre les événements de logon (**Event ID 4624** = succès) et les logons avec identifiants explicites (**Event ID 4648**, vu à la page 1) survenus **après** cette heure, dans le même fichier de logs :
   ```powershell
   Get-WinEvent -Path "C:\Logs\Dump\NOM_DU_FICHIER.evtx" -FilterXPath "*[System[(EventID=4624 or EventID=4648)]]" |
     Where-Object { $_.TimeCreated -gt (Get-Date "HEURE_DU_DUMP") } |
     Select-Object TimeCreated, Id, Message | Format-List
   ```
3. Examine chaque résultat :
   - Un **Logon Type inhabituel** (ex. Type 3 réseau ou Type 10 RDP) avec un compte qui n'est **pas** celui qui a lancé le dump
   - Un **Logon Server** ou une **IP source** externe/inattendue
   - Un compte à **privilèges élevés** (Administrator, compte de service) qui se connecte juste après le dump
   
   → sont des indices d'un logon malveillant, probablement avec les identifiants volés (attaque de type **pass-the-hash** ou réutilisation du mot de passe en clair).
4. Si un tel événement existe (connexion suspecte peu après le dump), la réponse est **Yes**. Sinon, **No**.

**Partie 3 : validation**

5. Saisis `Yes` ou `No` (selon ce que tu observes) sur HTB → **Submit**.

---

### ❓ Question 6 — Processus utilisé pour exécuter du code via une relation parent-enfant anormale

| | |
|---|---|
| **Question (EN)** | By examining the logs located in the "C:\Logs\StrangePPID" directory, determine a process that was used to temporarily execute code based on a strange parent-child relationship. Enter the process name as your answer. Answer format: _.exe |
| **Question (FR)** | En examinant les logs du dossier `C:\Logs\StrangePPID`, détermine un processus utilisé pour exécuter du code **temporairement**, à partir d'une relation parent-enfant anormale. |
| **Réponse à mettre sur HTB** | `*(à compléter)*` |

**🧭 Procédure complète**

**Partie 0 : connexion** (si pas déjà fait)

**Partie 1 : chercher les créations de processus suspectes (Event ID 1)**

1. **Event Viewer → Open Saved Log...** → sélectionne le(s) `.evtx` dans `C:\Logs\StrangePPID`.
2. Filtre sur **Event ID 1** (Process Creation) :
   ```powershell
   Get-WinEvent -Path "C:\Logs\StrangePPID\NOM_DU_FICHIER.evtx" -FilterXPath "*[System[(EventID=1)]]" |
     Select-Object TimeCreated, Message | Format-List
   ```
3. Pour chaque événement, compare le champ **ParentImage** au champ **Image** : cherche une relation **improbable** dans un environnement Windows standard (comme l'exemple du cours : `spoolsv.exe` → `cmd.exe`, ou `spoolsv.exe` → `whoami.exe`, au lieu de son `conhost.exe` habituel).
4. Repère le processus **enfant** lancé de façon anormale par un parent inattendu (ex. un process de service Windows qui lance un interpréteur de commandes ou un utilitaire de reconnaissance).

**Partie 2 : vérifier via ETW si disponible (comme à la page 4)**

5. Si un fichier de trace ETW (`.json` type SilkETW, ou un export du provider `Microsoft-Windows-Kernel-Process`) est présent dans le dossier, compare le **vrai** parent indiqué par ETW à celui affiché par Sysmon : une différence confirme un **Parent PID Spoofing**, comme dans l'exemple du cours.
6. Le processus qui a servi à **exécuter du code temporairement** via cette relation anormale (l'enfant détourné, lancé puis généralement arrêté rapidement) est ta réponse.

**Partie 3 : validation**

7. Le nom de l'exécutable trouvé (format `_.exe`) est ta réponse. Soumets-le sur HTB.

---

# Mémo final — toutes les réponses HTB

| Page | Question | **Réponse à mettre sur HTB** |
|---|---|---|
| 1 | Exécutable responsable de la modification d'audit | `TiWorker.exe` |
| 1 | Heure de modification d'audit sur wpfgfx_v0400.dll | `10:23:50` |
| 2 | Hash SHA256 de WININET.dll malveillante | `51F2305DCF385056C68F7CCF5B1B3B9304865CEF1257947D4AD6EF5FAD2E3B13` |
| 2 | Hash SHA256 de clrjit.dll (injection PowerShell) | `8A3CD3CF2249E9971806B15C75A892E6A44CCA5FF5EA5CA89FDA951CD2C09AA9` |
| 2 | Hash NTLM de l'Administrateur (credential dumping) | `5e4ffd54b3849aa720ed39f50185e533` |
| 3 | *(pas de question)* | — |
| 4 | ManagedInteropMethodName (G…ion) | `GetTokenInformation` |
| 5 | Heure d'ajout du partage \*\PRINT | `12:30:30` |
| 6 | Processus du DLL Hijacking (`C:\Logs\DLLHijack`) | *(à compléter)* |
| 6 | Processus exécutant le PowerShell non managé (`C:\Logs\PowershellExec`) | *(à compléter)* |
| 6 | Processus injecteur (`C:\Logs\PowershellExec`) | *(à compléter)* |
| 6 | Processus du dump LSASS (`C:\Logs\Dump`) | *(à compléter)* |
| 6 | Login malveillant après le dump ? (`C:\Logs\Dump`) | *(à compléter — Yes/No)* |
| 6 | Processus via relation parent-enfant anormale (`C:\Logs\StrangePPID`) | *(à compléter)* |

---

# Cheat-sheet du module

## Connexion & outils

| Objectif | Commande |
|---|---|
| RDP vers la cible | `xfreerdp /u:Administrator /p:'HTB_@cad3my_lab_W1n10_r00t!@0' /v:IP_CIBLE /dynamic-resolution` |
| Installer Sysmon | `sysmon.exe -i -accepteula -h md5,sha256,imphash -l -n` |
| Appliquer une config Sysmon | `sysmon.exe -c fichier.xml` |
| Hash d'un fichier (PowerShell) | `Get-FileHash "chemin" -Algorithm SHA256` |

## Event IDs à retenir

| Event ID | Source | Signification |
|---|---|---|
| **1** | Sysmon | Process Creation |
| **3** | Sysmon | Network Connection |
| **7** | Sysmon | Image Loaded (DLL hijack, injection .NET) |
| **10** | Sysmon | Process Access (LSASS dump, injection) |
| **1074** | Windows System | Shutdown/Restart |
| **1102** | Windows Security | Audit log cleared |
| **4624 / 4625** | Windows Security | Logon réussi / échoué |
| **4648** | Windows Security | Logon avec identifiants explicites |
| **4672** | Windows Security | Privilèges spéciaux attribués |
| **4698-4702** | Windows Security | Tâches planifiées (persistance) |
| **4719** | Windows Security | Politique d'audit modifiée |
| **4907** | Windows Security | SACL d'un objet modifiée |
| **5140 / 5142 / 5145** | Windows Security | Accès / création / vérification de partage réseau |
| **7045** | Windows Security | Service installé |

## logman.exe (ETW)

| Objectif | Commande |
|---|---|
| Lister les sessions de traçage actives | `logman.exe query -ets` |
| Détails d'une session | `logman.exe query "NomSession" -ets` |
| Lister tous les providers | `logman.exe query providers` |
| Filtrer les providers par mot-clé | `logman.exe query providers \| findstr "MotClé"` |
| Détails d'un provider précis | `logman.exe query providers NomDuProvider` |

## SilkETW

| Objectif | Commande |
|---|---|
| Capturer un provider vers un fichier JSON | `SilkETW.exe -t user -pn NomProvider -ot file -p C:\chemin\sortie.json` |
| Capturer avec un masque de keywords | `SilkETW.exe -t user -pn NomProvider -uk 0xMASQUE -ot file -p C:\chemin\sortie.json` |

## Get-WinEvent

| Objectif | Commande |
|---|---|
| Lister tous les logs | `Get-WinEvent -ListLog *` |
| Lister tous les providers | `Get-WinEvent -ListProvider *` |
| N événements d'un log | `Get-WinEvent -LogName 'NomLog' -MaxEvents N` |
| Événements les plus anciens | `Get-WinEvent -LogName 'NomLog' -Oldest -MaxEvents N` |
| Depuis un fichier .evtx | `Get-WinEvent -Path 'chemin.evtx'` |
| Filtrer par log + ID(s) | `Get-WinEvent -FilterHashtable @{LogName='NomLog'; ID=1,3}` |
| Filtrer par plage de dates | `Get-WinEvent -FilterHashtable @{LogName='NomLog'; StartTime=$d1; EndTime=$d2}` |
| Filtrer avec XPath | `Get-WinEvent -LogName 'NomLog' -FilterXPath "*[System[EventID=3] and EventData[Data[@Name='DestinationIp']='IP']]"` |
| Toutes les propriétés d'un événement | `Get-WinEvent ... \| Select-Object -Property *` |
| Filtrer sur une propriété indexée | `Get-WinEvent ... \| Where-Object {$_.Properties[N].Value -like "*motif*"}` |

**Index utile pour Sysmon Event ID 1** : `Properties[21]` = `ParentCommandLine`.