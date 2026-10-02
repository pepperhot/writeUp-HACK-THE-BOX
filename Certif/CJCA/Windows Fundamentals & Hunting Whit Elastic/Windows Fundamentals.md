# CJCA — Windows Fundamentals

## 1. Systèmes de fichiers

Windows utilise notamment :

* **FAT32** → clé USB, cartes SD, fichiers < 4 Go
* **NTFS** → système principal de Windows, permissions et journalisation
* **exFAT** → supports amovibles

👉 Le module se concentre principalement sur **NTFS**.

---

## 2. Permissions NTFS

### Permissions principales

* **F** → Full Control
* **M** → Modify
* **RX** → Read & Execute
* **R** → Read
* **W** → Write
* **D** → Delete
* **N** → No access

### Héritage

Les fichiers et dossiers héritent généralement des permissions du dossier parent.

### icacls

Permet de voir et modifier les permissions NTFS.

```cmd
icacls C:\Users
icacls C:\Users /grant joe:F
icacls C:\Users /remove joe
```

Pour appliquer aux fichiers et sous-dossiers :

```cmd
(OI)(CI)
```

* **OI** → fichiers héritent
* **CI** → dossiers héritent

---

## 3. NTFS vs SMB

**SMB** permet de partager des fichiers et imprimantes sur le réseau.

### Permissions SMB

* **Full Control**
* **Change**
* **Read**

Sur un partage SMB :

> **Permissions SMB + permissions NTFS → la plus restrictive s'applique.**

Les permissions NTFS s'appliquent également en local et via RDP.

### Partages administratifs

```text
C$      → disque C:
ADMIN$  → C:\Windows
IPC$    → communication inter-processus
```

### Commandes

```cmd
net share
```

Depuis Linux :

```bash
smbclient -L SERVER_IP -U user
smbclient '\\SERVER_IP\Share' -U user
```

---

## 4. Registre Windows

Base de données contenant la configuration de Windows.

```text
Ruche
 ↓
Clé
 ↓
Sous-clé
 ↓
Valeur
```

### Ruches principales

| Ruche    | Rôle                     |
| -------- | ------------------------ |
| **HKCU** | Utilisateur actuel       |
| **HKLM** | Système                  |
| **HKCR** | Associations de fichiers |
| **HKU**  | Utilisateurs             |
| **HKCC** | Configuration matérielle |

### Types à connaître

* **REG_SZ** → texte
* **REG_DWORD** → nombre 32 bits
* **REG_QWORD** → nombre 64 bits
* **REG_BINARY** → données binaires

### Commandes

```cmd
reg query "HKCU\Software\..."
reg add "HKCU\Software\..." ...
reg delete "HKCU\Software\..." ...
```

### Persistance

Les clés `Run` et `RunOnce` permettent de lancer des programmes au démarrage ou à la connexion.

```text
HKCU\...\CurrentVersion\Run
HKLM\...\CurrentVersion\Run
```

---

## 5. Services et processus

### Services

Les services tournent en arrière-plan et sont gérés par le **Service Control Manager (SCM)**.

États :

* Running
* Stopped
* Paused

Commandes :

```powershell
Get-Service
Get-CimInstance Win32_Service | select Name,PathName
```

⚠️ Une mauvaise configuration d'un service peut permettre une **élévation de privilèges**.

### Processus importants

* **lsass.exe** → authentification et sécurité
* **services.exe** → gestion des services
* **winlogon.exe** → connexion Windows
* **svchost.exe** → héberge des services
* **System** → processus noyau

**LSASS** est une cible importante car des identifiants peuvent être présents dans sa mémoire.

---

## 6. PowerShell et CMD

### CMD

```cmd
help
help schtasks
ipconfig /?
```

### PowerShell

Utilise des commandes sous la forme :

```text
Verbe-Nom
```

Exemples :

```powershell
Get-ChildItem
Get-Service
Get-Alias
Get-Help <commande>
```

Quelques alias :

```text
ls / gci → Get-ChildItem
cd / sl  → Set-Location
cat      → Get-Content
?        → Where-Object
%        → ForEach-Object
```

---

## 7. WMI

**WMI (Windows Management Instrumentation)** permet de gérer et surveiller Windows.

Utilisations :

* informations système ;
* gestion à distance ;
* sécurité ;
* permissions ;
* exécution de code ;
* déplacement latéral.

Exemples :

```cmd
wmic os get serialnumber
wmic computersystem get name
```

```powershell
Get-WmiObject -Class Win32_OperatingSystem
```

---

## 8. Sécurité Windows

### SID

Chaque utilisateur, groupe ou ordinateur possède un **SID** unique.

```cmd
whoami /user
```

### ACL / ACE

* **ACE** → entrée de permission
* **ACL** → ensemble d'ACE
* **DACL** → contrôle qui a accès
* **SACL** → contrôle l'audit

### UAC

**User Account Control** empêche l'exécution silencieuse d'actions administratives.

---

## 9. Protection Windows

### AppLocker

Permet de contrôler quels programmes peuvent être exécutés.

Règles possibles :

* éditeur ;
* fichier ;
* chemin ;
* hash ;
* version.

### Windows Defender

Protection notamment contre :

* malware ;
* ransomware ;
* modifications malveillantes.

Commande :

```powershell
Get-MpComputerStatus
```

---

## 10. Commandes à retenir

| Objectif            | Commande                        |
| ------------------- | ------------------------------- |
| Permissions         | `icacls <chemin>`               |
| Partages            | `net share`                     |
| Registre            | `reg query`                     |
| Services            | `Get-Service`                   |
| Chemin d'un service | `Get-CimInstance Win32_Service` |
| Alias               | `Get-Alias`                     |
| Execution Policy    | `Get-ExecutionPolicy -List`     |
| SID utilisateur     | `whoami /user`                  |
| Numéro de série     | `wmic os get serialnumber`      |
| Defender            | `Get-MpComputerStatus`          |

---

## ⚠️ À retenir

> **NTFS = permissions fichiers/dossiers**

> **SMB = partage réseau**

> **Sur SMB, la permission la plus restrictive s'applique.**

> **icacls = gestion des permissions NTFS**

> **Registre = configuration Windows**

> **LSASS = authentification / informations de sécurité**

> **WMI = gestion et surveillance Windows**

> **SID = identifiant unique de sécurité**

> **DACL = accès / SACL = audit**

> **UAC = protection contre les actions administratives silencieuses**
