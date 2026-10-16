# 🎯 Fiche de révision HTB — Bug Bounty Hunting Process

> **Légende**
> - 📘 Résumé du cours (traduit en français)
> - 🔑 À retenir absolument
> - ⌨️ Éléments / termes importants
> - ✅ Question, réponse en français, et **Réponse à mettre sur HTB** (telle qu'attendue)
> - 🧭 Procédure détaillée, étape par étape, pour retrouver la réponse
> - ⚠️ Passage reconstitué (absent du cours) ou réponse manquante à compléter

---

## Sommaire

1. [Page 1 — Bug Bounty Programs](#page-1--bug-bounty-programs)
2. [Page 2 — Writing a Good Report](#page-2--writing-a-good-report)
3. [Page 3 — Interacting with Organizations/BBP Hosts](#page-3--interacting-with-organizationsbbp-hosts)
4. [Page 4 — Example 1 : Reporting Stored XSS](#page-4--example-1--reporting-stored-xss)
5. [Page 5 — Example 2 : Reporting CSRF](#page-5--example-2--reporting-csrf)
6. [Page 6 — Example 3 : Reporting RCE](#page-6--example-3--reporting-rce)
7. [Mémo final : toutes les réponses HTB](#mémo-final--toutes-les-réponses-htb)
8. [Cheat-sheet : structure d'un bug bounty program et d'un bon rapport](#cheat-sheet--structure-dun-bug-bounty-program-et-dun-bon-rapport)
9. [Cheat-sheet CVSS 3.1 (Base Score)](#cheat-sheet-cvss-31-base-score)

> ℹ️ Ce module est **conceptuel/méthodologique** : il n'y a pas de cible à spawn ni de payload à exécuter. Les questions se répondent directement à partir du texte du cours.

---

# Page 1 — Bug Bounty Programs

## 📘 Résumé

Un **bug bounty program** (BBP), aussi appelé **vulnerability rewards program (VRP)**, est une démarche de test de sécurité continue et proactive qui vient compléter les audits de code internes et les pentests classiques, dans le cadre global de la gestion des vulnérabilités d'une organisation. HackerOne la décrit comme « continuous testing, constant protection ».

## 🔑 À retenir

### Types de programmes

| Type | Description |
|---|---|
| **Privé** | Accessible uniquement sur invitation. La majorité des BBP démarrent privés, le temps que l'organisation rode son processus de triage, puis deviennent publics. Les invitations dépendent du historique du hunter (qualité et constance des findings, absence de violations) ; certains programmes exigent même un background check. |
| **Public** | Ouvert à toute la communauté de hackers. |
| **Parent/Child** | Une entreprise mère et ses filiales partagent une même cagnotte (bounty pool) et une même équipe sécurité ; le programme d'une filiale (child) est relié à celui de la maison mère (parent). |

### BBP vs VDP — ne pas confondre

| | Bug Bounty Program (BBP) | Vulnerability Disclosure Program (VDP) |
|---|---|---|
| Définition | Incite activement des tiers à trouver et signaler des bugs | Donne juste des indications sur comment l'organisation préfère recevoir les signalements |
| Récompense | Rémunération monétaire | Pas de récompense financière |

### Code de conduite

Le **violation record** d'un hunter est toujours pris en compte. Il faut lire attentivement le code de conduite / la policy de chaque programme ou plateforme : ça établit les attentes de comportement et rend aussi les hunters plus efficaces dans la soumission de leurs rapports. HackerOne a son propre Code of Conduct de référence.

### Structure d'un programme (section « Policy »)

| Élément | Rôle |
|---|---|
| **Vendor Response SLAs** | Définit quand et comment le vendeur répondra |
| **Access** | Comment créer/obtenir des comptes pour la recherche |
| **Eligibility Criteria** | Ex. être le premier à rapporter une vulnérabilité pour être éligible |
| **Responsible Disclosure Policy** | Délais de divulgation, coordination pour divulguer en sécurité |
| **Rules of Engagement** | Règles d'engagement |
| **Scope** | IP, domaines, vulnérabilités dans le périmètre |
| **Out of Scope** | IP, domaines, vulnérabilités hors périmètre |
| **Reporting Format** | Format attendu du rapport |
| **Rewards** | Barème des récompenses |
| **Safe Harbor** | Garanties légales pour le hunter de bonne foi |
| **Legal Terms and Conditions** | Conditions légales |
| **Contact Information** | Coordonnées de contact |

### Trouver des programmes

Le meilleur point de départ : le **Directory de HackerOne**, qui liste les organisations avec BBP et leurs informations de contact.

## ✅ Questions

### ❓ Question 1 — Élément qui établit les attentes de comportement

| | |
|---|---|
| **Question (EN)** | Which Bug Bounty Program part establishes expectations for behavior while participating in a bug bounty program? |
| **Question (FR)** | Quel élément du Bug Bounty Program établit les attentes de comportement durant la participation au programme ? |
| **Réponse (FR)** | Le cours précise explicitement que lire la policy et le **Code of Conduct** « establishes expectations for behavior » — c'est le document qui fixe ces attentes. |
| **Réponse à mettre sur HTB** | `Code of Conduct` |

**🧭 Procédure complète**

1. Relis le paragraphe « Bug Bounty Program Code of Conduct » de la page.
2. Repère la phrase clé : le code de conduite/policy « does not just establish expectations for behavior but also makes bug bounty hunters more effective ».
3. C'est une citation quasi-directe de la réponse attendue.
4. Saisis `Code of Conduct` → **Submit task answer**.

✔️ **Résultat attendu** : réponse acceptée telle quelle (respecte la casse du terme usuel « Code of Conduct »).

---

# Page 2 — Writing a Good Report

## 📘 Résumé

Un bon rapport de bug doit être clair, concis, et surtout **reproductible étape par étape**. Face à des entreprises moins matures en sécurité, il faut parfois traduire l'impact technique en termes business compréhensibles.

## 🔑 À retenir

### Éléments essentiels d'un bon rapport (l'ordre peut varier)

| Élément | Rôle |
|---|---|
| **Vulnerability Title** | Type de vulnérabilité, domaine/paramètre/endpoint affecté, impact, etc. |
| **CWE & CVSS score** | Communique la nature et la sévérité de la vulnérabilité |
| **Vulnerability Description** | Explique la cause de la vulnérabilité |
| **Proof of Concept (POC)** | Étapes claires et concises pour reproduire l'exploitation |
| **Impact** | Ce qu'un attaquant peut accomplir ; impact business et dommage maximal |
| **Remediation** | Optionnel en bug bounty, mais apprécié |

Un rapport bien formaté réduit fortement le temps de reproduction et de triage.

### Pourquoi CWE & CVSS ?

- **CWE (Common Weaknesses Enumeration)** : liste communautaire de types de faiblesses logicielles/matérielles — un langage commun, une référence pour les outils de sécurité, une base pour l'identification/mitigation/prévention. En cas de chaîne de vulnérabilités, choisir le CWE lié à la vulnérabilité **initiale**.
- **CVSS (Common Vulnerability Scoring System)** : standard publié et utilisé mondialement pour communiquer la **sévérité**.

### CVSS 3.1 — Base Score (métriques détaillées)

| Métrique | Valeurs | Signification |
|---|---|---|
| **Attack Vector** | Network (N) / Adjacent (A) / Local (L) / Physical (P) | N = exploitable à distance via le réseau · **A = l'attaquant doit être sur le même réseau physique/logique que la cible (y compris VPN sécurisé)** · L = accès local (clavier, SSH...) ou interaction utilisateur · P = interaction physique directe |
| **Attack Complexity** | Low (L) / High (H) | L = pas de préparation spéciale, répétable facilement · H = préparation et reconnaissance nécessaires |
| **Privileges Required** | None (N) / Low (L) / High (H) | N = aucun accès requis · L = privilèges utilisateur standard · H = privilèges admin |
| **User Interaction** | None (N) / Required (R) | N = exploitation autonome · R = la victime doit agir |
| **Scope** | Unchanged (U) / Changed (C) | U = n'affecte que le composant vulnérable (même autorité de sécurité) · C = affecte des composants au-delà de ce périmètre |
| **Confidentiality** | None / Low / High | Perte d'accès non autorisé à l'information |
| **Integrity** | None / Low / High | Altération non autorisée des données |
| **Availability** | None / Low / High | Perte de disponibilité du service |

### Exemples cités dans le cours

| CVE / Cas | Score | Vecteur clé |
|---|---|---|
| Cisco ASA IKEv1/IKEv2 Buffer Overflow (CVE-2016-1287) | 9.8 (Critical) | Network, Low complexity, No privileges, No interaction → reverse shell complet |
| Stored XSS dans un panneau admin (Malicious Admin → Admin) | 5.5 (Medium) | Network, Low complexity, **High privileges requis pour poster**, No interaction, **Scope Changed** (serveur → navigateur) |

### Bons rapports à consulter (exemples HackerOne cités)

- SSRF in Exchange leads to ROOT access in all instances
- Remote Code Execution in Slack desktop apps
- Full name of other accounts exposed through NR API Explorer
- A staff member with no permissions can edit Store Customer Email
- XSS while logging in using Google
- Cross-site Scripting (XSS) on HackerOne careers page

## ✅ Questions

### ❓ Question 1 — Métrique « Adjacent »

| | |
|---|---|
| **Question (EN)** | Which base metric value of the base score considers that attackers can only exploit a vulnerability if they reside in the same physical or logical network as the target host/application? |
| **Question (FR)** | Quelle valeur de métrique de base du Base Score considère que l'attaquant ne peut exploiter la vulnérabilité que s'il se trouve sur le même réseau physique ou logique que la cible ? |
| **Réponse (FR)** | C'est la définition exacte de **Adjacent (A)** dans la métrique **Attack Vector**, qui inclut explicitement les VPN sécurisés. |
| **Réponse à mettre sur HTB** | `Adjacent` |

**🧭 Procédure complète**

1. Repère la section « Attack Vector » dans le descriptif du CVSS Calculator.
2. Compare les 4 valeurs : Network (réseau global/distant), **Adjacent** (même réseau physique/logique, VPN inclus), Local (accès local/SSH/interaction), Physical (manipulation physique).
3. La formulation de la question reprend mot pour mot la définition d'**Adjacent (A)**.
4. Saisis `Adjacent` → **Submit task answer**.

✔️ **Résultat attendu** : réponse acceptée telle quelle.

---

# Page 3 — Interacting with Organizations/BBP Hosts

## 📘 Résumé

Comment se comporter après l'envoi d'un rapport ? La règle d'or : **patienter**. Laisser le temps à l'équipe triage de traiter, valider, et éventuellement poser des questions.

## 🔑 À retenir

- Les **Vendor Response SLAs** (ou les métriques d'efficacité de réponse du programme) donnent une idée du délai normal de réponse — s'y référer avant de s'impatienter.
- Ne jamais **spammer** l'équipe triage sur une courte période.
- Si pas de retour dans un délai raisonnable **et** que la soumission passait par une plateforme (ex. HackerOne) → contacter la **Médiation** (Hacker Mediation).
- Une fois qu'un membre de l'équipe triage répond : **noter son username** et le **taguer** dans les communications suivantes (il traitera probablement le dossier).
- **Jamais** d'interaction via un canal non officiel (réseaux sociaux, etc.) — rester sur les canaux officiels du programme/de la plateforme.
- Communication professionnelle = rapport professionnel. Rester calme, se comporter en professionnel de la sécurité.

### En cas de désaccord sur la sévérité ou le montant de la prime

Procéder dans cet ordre :

1. **Expliquer le raisonnement** : guider l'équipe triage métrique par métrique du calculateur CVSS utilisé, jusqu'à trouver un accord.
2. **Revérifier policy et scope** : s'assurer que la soumission est bien conforme au périmètre du programme, et que le montant proposé correspond à la policy du programme.
3. **Si rien n'a abouti** : contacter la médiation ou un service de plateforme équivalent.

## ✅ Questions

*(pas de question sur cette page)*

---

# Page 4 — Example 1 : Reporting Stored XSS

## 📘 Résumé

Exemple de rapport complet et réaliste, utilisé comme modèle à reproduire dans tes propres rapports. Vulnérabilité : un **nom de fichier uploadé** (filename) non sanitizé, à la fois réfléchi au navigateur et stocké en base → XSS **reflected ET stored**, exploitable entre administrateurs d'un panneau d'admin.

## 🔑 À retenir — structure du rapport modèle

| Section | Contenu |
|---|---|
| **Title** | « Stored Cross-Site Scripting (XSS) in X Admin Panel » |
| **CWE** | CWE-79 : Improper Neutralization of Input During Web Page Generation (XSS) |
| **CVSS 3.1 Score** | 5.5 (Medium) |
| **Description** | Mécanisme d'upload de fichier (« Admin Info » → « Secure Data Transfer » → « Load of Data ») : le filename n'est pas sanitizé, réfléchi ET stocké en base |
| **Impact** | Vol d'identifiants, session hijacking, defacement, redirection malveillante ; ici aggravé car **tous les admins** voient les fichiers uploadés par n'importe qui |
| **POC — Étape 1** | Un admin malveillant nomme son fichier uploadé avec un payload JS, ex. : `"><svg onload = alert(document.cookie)>.docx` |
| **POC — Étape 2** | Quand un autre admin clique sur « view » pour ouvrir ce fichier, le payload s'exécute dans son navigateur |

## ⌨️ Payload important

```html
"><svg onload = alert(document.cookie)>.docx
```
→ `">` sert à **échapper** un attribut/une balise HTML existante avant d'injecter ; `<svg onload=...>` est un vecteur classique sans `<script>` (cf. module XSS) ; `.docx` conserve une apparence de nom de fichier légitime.

## 🔑 Breakdown CVSS appliqué à ce cas

| Métrique | Valeur | Justification |
|---|---|---|
| Attack Vector | Network | Attaque possible via internet |
| Attack Complexity | Low | Il suffit de choisir le bon payload dans le filename |
| Privileges Required | **High** | Il faut déjà être admin pour accéder au panneau et uploader |
| User Interaction | None | Les autres admins sont touchés en naviguant normalement sur une page régulièrement visitée |
| Scope | **Changed** | Composant vulnérable = serveur web, composant impacté = navigateur |
| Confidentiality | Low | Accès possible au DOM |
| Integrity | Low | XSS permet d'affecter légèrement l'intégrité |
| Availability | None | Pas de déni de service possible via XSS |

## ✅ Questions

*(pas de question sur cette page — page d'exemple à étudier comme modèle de rapport)*

---

# Page 5 — Example 2 : Reporting CSRF

## 📘 Résumé

Deuxième exemple de rapport : une **Cross-Site Request Forgery (CSRF)** sur un formulaire d'enregistrement de consommateur (consumer registration). Aucune protection anti-CSRF (ex. token) n'était en place sur la requête de création d'application fintech.

## 🔑 À retenir — structure du rapport modèle

| Section | Contenu |
|---|---|
| **Title** | « Cross-Site Request Forgery (CSRF) in Consumer Registration » |
| **CWE** | CWE-352 : Cross-Site Request Forgery (CSRF) |
| **CVSS 3.1 Score** | 5.4 (Medium) |
| **Description** | CSRF = forcer la victime à charger une page qui exécute une requête malveillante avec son identité/privilèges, sans qu'elle le sache |
| **Impact** | L'attaquant peut effectuer toute opération à la place de la victime ; ici : enregistrer une application fintech et créer une clé API au nom de la victime |
| **POC — Étape 1** | Interception via proxy de la requête de création d'application → aucune protection anti-CSRF détectée |
| **POC — Étape 2** | Construction d'une page HTML malveillante qui reproduit cette requête (auto-soumission de formulaire) |
| **POC — Étape 3** | Envoi de la page à une victime avec une session active |
| **POC — Étape 4** | Création involontaire d'une nouvelle application fintech au nom de la victime |

⚠️ Note importante du cours : cette attaque **aurait pu se faire en arrière-plan** si elle était combinée avec la faille **6.1.1** (une XSS) du même rapport — illustre qu'un rapport de pentest peut chaîner plusieurs vulnérabilités entre elles.

## 🔑 Breakdown CVSS appliqué à ce cas

| Métrique | Valeur | Justification |
|---|---|---|
| Attack Vector | Network | Attaque réalisable via internet |
| Attack Complexity | Low | Il suffit de piéger un utilisateur avec une session ouverte vers un site malveillant |
| Privileges Required | **None** | L'attaquant n'a besoin d'aucun privilège |
| User Interaction | **Required** | La victime doit cliquer sur le lien piégé |
| Scope | **Unchanged** | Composant vulnérable = serveur web, composant impacté = le même serveur web |
| Confidentiality | Low | L'attaquant obtient des informations limitées (création d'appli) |
| Integrity | Low | Modification de données limitée, sans affecter sérieusement l'intégrité globale |
| Availability | None | Pas de déni de service possible via cette CSRF |

## ✅ Questions

*(pas de question sur cette page — page d'exemple à étudier comme modèle de rapport)*

---

# Page 6 — Example 3 : Reporting RCE

## 📘 Résumé

Dernier exemple : une **Remote Code Execution (RCE)** critique sur un serveur **IBM WebSphere**, via **désérialisation Java non sécurisée** (insecure Java object deserialization), exploitant la bibliothèque **Apache Commons Collections (ACC)**.

## 🔑 À retenir — structure du rapport modèle

| Section | Contenu |
|---|---|
| **Title** | « IBM WebSphere Java Object Deserialization RCE » |
| **CWE** | CWE-502 : Deserialization of Untrusted Data |
| **CVSS 3.1 Score** | 9.8 (Critical) |
| **Description** | Requête sur le port 8880 (HTTPS) → objets Java sérialisés en base64 détectés (en-tête `rO0` typique des objets Java sérialisés encodés base64). Construction d'une requête SOAP contenant un objet sérialisé malveillant exploitant ACC |
| **Impact** | Injection de commande = exécution de commande arbitraire sous le contexte de sécurité de l'utilisateur courant ; si exécuté en tant que compte privilégié (ex. SYSTEM) → prise de contrôle totale du système |
| **POC — Étape 1** | Capture et décodage d'une requête vers le port 8880 → confirme l'usage d'objets Java sérialisés |
| **POC — Étape 2** | Construction d'une requête SOAP contenant une commande (ping) à exécuter sur le serveur distant |
| **POC — Étape 3** | Envoi de la requête SOAP forgée → capture Wireshark confirmant la réception du ping ICMP depuis le serveur WebSphere vers la machine de l'attaquant |

## 🔑 À retenir — indice technique clé

- Un objet Java sérialisé encodé en base64 se reconnaît à son **en-tête `rO0`** (c'est la signature binaire standard d'un flux `ObjectOutputStream` Java, encodée en base64).

## 🔑 Breakdown CVSS appliqué à ce cas

| Métrique | Valeur | Justification |
|---|---|---|
| Attack Vector | Network | Attaque réalisable via internet |
| Attack Complexity | Low | Il suffit d'envoyer une requête forgée |
| Privileges Required | None | Exploitation possible sans authentification |
| User Interaction | None | Aucune interaction utilisateur nécessaire |
| Scope | **Unchanged** | Composant vulnérable = serveur web, composant impacté = le même serveur web |
| Confidentiality | **High** | RCE → contrôle total de l'information obtenue |
| Integrity | **High** | RCE → modification possible de toutes les données critiques |
| Availability | **High** | RCE → possibilité d'éteindre le serveur, déni de service total |

## ✅ Questions

*(pas de question sur cette page — dernière page d'exemple, module purement théorique à ce stade)*

---

# Mémo final — toutes les réponses HTB

| Page | Question | **Réponse à mettre sur HTB** |
|---|---|---|
| 1 | Élément qui établit les attentes de comportement | `Code of Conduct` |
| 2 | Métrique considérant un attaquant sur le même réseau physique/logique | `Adjacent` |
| 3 | *(pas de question)* | — |
| 4 | *(pas de question)* | — |
| 5 | *(pas de question)* | — |
| 6 | *(pas de question)* | — |

---

# Cheat-sheet : structure d'un bug bounty program et d'un bon rapport

## Un Bug Bounty Program (section Policy)

Vendor Response SLAs · Access · Eligibility Criteria · Responsible Disclosure Policy · Rules of Engagement · Scope · Out of Scope · Reporting Format · Rewards · Safe Harbor · Legal Terms and Conditions · Contact Information

## Un bon rapport de bug

Vulnerability Title → CWE & CVSS score → Vulnerability Description → Proof of Concept (POC) → Impact → Remediation *(optionnel)*

## Règles d'interaction post-soumission

1. Patienter (voir les SLA du programme).
2. Ne jamais spammer.
3. Si retard anormal + plateforme type HackerOne → Médiation.
4. Noter et taguer le username du membre triage assigné.
5. Jamais de canal non officiel.
6. En cas de désaccord sur la sévérité : (a) réexpliquer métrique par métrique du CVSS → (b) revérifier policy/scope/bounty → (c) médiation en dernier recours.

---

# Cheat-sheet CVSS 3.1 (Base Score)

| Métrique | Valeurs possibles |
|---|---|
| **Attack Vector (AV)** | Network (N) · Adjacent (A) · Local (L) · Physical (P) |
| **Attack Complexity (AC)** | Low (L) · High (H) |
| **Privileges Required (PR)** | None (N) · Low (L) · High (H) |
| **User Interaction (UI)** | None (N) · Required (R) |
| **Scope (S)** | Unchanged (U) · Changed (C) |
| **Confidentiality (C)** | None · Low · High |
| **Integrity (I)** | None · Low · High |
| **Availability (A)** | None · Low · High |

## Cas types vus dans le cours

| Vulnérabilité | Score | Profil typique |
|---|---|---|
| Buffer Overflow réseau non authentifié (ex. Cisco ASA CVE-2016-1287) | 9.8 Critical | AV:N / AC:L / PR:N / UI:N / S:U / C:H / I:H / A:H |
| Stored XSS en panneau admin (nécessite déjà d'être admin) | 5.5 Medium | AV:N / AC:L / PR:H / UI:N / S:C / C:L / I:L / A:N |
| CSRF sur formulaire d'enregistrement | 5.4 Medium | AV:N / AC:L / PR:N / UI:R / S:U / C:L / I:L / A:N |
| Désérialisation Java non authentifiée → RCE | 9.8 Critical | AV:N / AC:L / PR:N / UI:N / S:U / C:H / I:H / A:H |

**À retenir pour la logique du scoring** :
- **Scope = Changed** dès que le composant *impacté* par l'exploitation diffère du composant *vulnérable* (ex. XSS : serveur vulnérable, navigateur impacté).
- **Privileges Required = High** dès qu'il faut déjà être admin pour déclencher l'attaque (même si la victime est un autre admin).
- Un **RCE non authentifié** atteint quasi systématiquement 9.8 (C/I/A tous à High, AV:N, AC:L, PR:N, UI:N).