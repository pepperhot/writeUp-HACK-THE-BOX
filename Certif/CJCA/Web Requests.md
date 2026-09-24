# HTB Academy - Web Requests

## 1. HTTP : requête et réponse

HTTP fonctionne avec :

* **Requête** → envoyée par le client (navigateur, cURL)
* **Réponse** → envoyée par le serveur

Une requête contient principalement :

```text
GET /index.html HTTP/1.1
Host: example.com
User-Agent: ...
```

Une réponse contient un **code HTTP**, des en-têtes et éventuellement un corps.

Quelques codes importants :

| Code      | Signification               |
| --------- | --------------------------- |
| `200`     | OK                          |
| `301/302` | Redirection                 |
| `400`     | Mauvaise requête            |
| `401`     | Authentification nécessaire |
| `403`     | Accès interdit              |
| `404`     | Ressource inexistante       |
| `500`     | Erreur serveur              |

### cURL

```bash
curl http://IP:PORT -v
curl http://IP:PORT -vvv
```

`>` = requête envoyée
`<` = réponse reçue

---

## 2. En-têtes HTTP

Les headers donnent des informations sur la requête ou la réponse.

### Headers importants

| Header             | Rôle                                |
| ------------------ | ----------------------------------- |
| `Host`             | Serveur ciblé                       |
| `User-Agent`       | Client utilisé                      |
| `Referer`          | Page d'origine                      |
| `Accept`           | Types de données acceptés           |
| `Cookie`           | Session utilisateur                 |
| `Authorization`    | Authentification                    |
| `Content-Type`     | Type des données envoyées           |
| `Content-Length`   | Taille du contenu                   |
| `Server`           | Logiciel du serveur                 |
| `Set-Cookie`       | Création/modification d'un cookie   |
| `WWW-Authenticate` | Méthode d'authentification demandée |

### Headers de sécurité

```text
Content-Security-Policy
Strict-Transport-Security
Referrer-Policy
```

Ils permettent notamment de limiter certaines attaques web.

### Modifier les headers avec cURL

```bash
curl -I http://site
curl -i http://site
curl -A 'Mozilla/5.0' http://site
curl -H 'Nom: valeur' http://site
```

---

## 3. GET

GET permet de récupérer une ressource. Les paramètres sont généralement placés dans l'URL :

```text
/search.php?search=london
```

Exemple :

```bash
curl 'http://IP:PORT/search.php?search=london'
```

### HTTP Basic Authentication

Une réponse `401` avec :

```text
WWW-Authenticate: Basic
```

indique généralement une authentification Basic.

```bash
curl -u admin:admin http://IP:PORT/
```

On peut également envoyer directement le header :

```bash
curl -H 'Authorization: Basic BASE64' http://IP:PORT/
```

Le Base64 encode `username:password`, mais **ne chiffre pas** les identifiants.

---

## 4. POST

POST permet d'envoyer des données dans le **corps de la requête**.

Exemple de formulaire :

```bash
curl -X POST \
-d 'username=admin&password=admin' \
http://IP:PORT/
```

Pour suivre une redirection :

```bash
curl -L -X POST -d '...' http://IP:PORT/
```

### Cookies

Après une connexion, le serveur peut renvoyer :

```text
Set-Cookie: PHPSESSID=...
```

Réutiliser le cookie :

```bash
curl -b 'PHPSESSID=valeur' http://IP:PORT/
```

Ou utiliser un fichier de cookies :

```bash
curl -c cookies.txt -X POST -d '...' http://IP:PORT/
curl -b cookies.txt http://IP:PORT/
```

### POST JSON

```bash
curl -X POST \
-d '{"search":"london"}' \
-H 'Content-Type: application/json' \
http://IP:PORT/search.php
```

Pour une requête JSON, le `Content-Type` est important.

---

## 5. API REST et CRUD

Une API REST utilise les méthodes HTTP pour manipuler des ressources.

| Opération | Méthode         |
| --------- | --------------- |
| Create    | `POST`          |
| Read      | `GET`           |
| Update    | `PUT` / `PATCH` |
| Delete    | `DELETE`        |

Exemple :

```text
/api.php/city/london
```

### GET

```bash
curl -s http://IP:PORT/api.php/city/london | jq
curl -s http://IP:PORT/api.php/city/ | jq
curl -s http://IP:PORT/api.php/city/ | jq length
```

### POST

```bash
curl -X POST http://IP:PORT/api.php/city/ \
-d '{"city_name":"HTB_City","country_name":"HTB"}' \
-H 'Content-Type: application/json'
```

### PUT

```bash
curl -X PUT http://IP:PORT/api.php/city/london \
-d '{"city_name":"New_HTB_City","country_name":"HTB"}' \
-H 'Content-Type: application/json'
```

### DELETE

```bash
curl -X DELETE http://IP:PORT/api.php/city/New_HTB_City
```

`jq` permet de lire facilement les réponses JSON :

```bash
| jq
| jq length
```

---

## 6. DevTools

Les DevTools permettent d'observer les requêtes réellement envoyées par le navigateur.

**F12 → Network → Recharger la page**

Pour analyser une requête :

1. Trouver la requête intéressante.
2. Vérifier la méthode (`GET`, `POST`...).
3. Vérifier l'URL.
4. Vérifier les headers.
5. Vérifier les cookies.
6. Vérifier le corps de la requête.
7. Lire la réponse.

Une requête peut être copiée directement :

```text
Clic droit → Copy → Copy as cURL
```

Cela permet ensuite de la rejouer et de modifier ses paramètres.

---
