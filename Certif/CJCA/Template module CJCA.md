# 📋 TEMPLATE À COLLER AU DÉBUT D'UNE NOUVELLE CONVERSATION

> Mode d'emploi : ouvre une conversation vierge, colle **tout ce fichier** (instructions + exemple), remplace uniquement le nom du module à la ligne « MODULE À TRAITER », puis envoie les pages du cours.

---

## 🎯 TON RÔLE ET MON OBJECTIF

Je prépare la certification **HTB CJCA** (Hack The Box, niveau junior) et j'ai terminé tous les modules. Je veux une **fiche de révision complète par module**, en **français**, au format **Markdown**, très lisible et facile à comprendre.

**MODULE À TRAITER : `[NOM DU MODULE EN ANGLAIS, tel qu'écrit sur HTB]`**

---

## 📥 COMMENT JE T'ENVOIE LE COURS

- Je t'envoie le contenu **page par page** (cours + questions + réponses de la page), parfois plusieurs pages dans un seul message.
- Je t'indique toujours quelle est **la dernière page**. Pour les questions que je valide sur HTB, je te donne les **réponses finales** : elles sont fiables et tu les utilises telles quelles.
- Pour chaque message de pages, réponds seulement : « Reçu, pages X à Y. » Ne génère pas encore la fiche.
- Quand j'écris **« FIN DU MODULE »** (ou quand j'envoie tout avec la dernière page indiquée), tu génères la fiche complète en **un seul fichier `.md`** que tu me présentes en téléchargement.

---

## 📐 FORMAT OBLIGATOIRE DE LA FICHE

### Structure globale
1. Titre : `# 🎯 Fiche de révision HTB CJCA — [Nom du module]`
2. Une **légende** (📘 Résumé, 🔑 À retenir, ⌨️ Commandes, ✅ Questions, 🧭 Procédure, ⚠️ Reconstitué)
3. Un **sommaire** avec liens vers chaque page
4. Si le module contient des questions sur une **cible** (lab, machine, Pwnbox, VPN) : un **🔌 Bloc de connexion** complet en haut de la fiche (voir plus bas)
5. Une section par page du cours : `# Page N — [Titre de la page]`
6. Un **mémo final** : tableau de toutes les réponses HTB, page par page
7. Une **cheat-sheet** des commandes/requêtes importantes du module

### Pour chaque page
- **📘 Résumé** traduit en français : tableaux, listes courtes, schémas ASCII quand ça aide. Explications simples, avec un exemple concret quand c'est utile.
- **🔑 À retenir** : les définitions et points qui tombent à l'examen.
- **⌨️ Commandes / requêtes importantes** : dans des blocs de code, avec une ligne d'explication. Pour les modules avec lab, ajoute le déroulé « étape → commande → ce qu'on découvre ».
- **✅ Questions** : une sous-section **par question** (voir le gabarit ci-dessous). Si la page n'a pas de question, écris-le.

### Gabarit obligatoire pour CHAQUE question

```
### ❓ Question N — [titre court]

| | |
|---|---|
| **Question (EN)** | texte exact de HTB |
| **Question (FR)** | traduction |
| **Réponse (FR)** | réponse expliquée en français |
| **Réponse à mettre sur HTB** | `réponse exacte, en anglais, telle que saisie` |

**🧭 Procédure complète**

**Partie 0 : connexion à la cible** (uniquement si la question demande une cible)
  numérotée, avec TOUTES les commandes, dans des blocs de code
**Partie 1 : la recherche / l'exploitation**
  étapes numérotées, une action par étape, commande exacte à copier-coller
**Partie 2 : validation**
  ce qu'on saisit sur HTB, avec la casse exacte

✔️ Résultat attendu / vérification
✔️ Plan B si ça ne marche pas
```

### Règles de la procédure détaillée (très important)
1. **Tout est écrit, rien n'est sous-entendu.** Chaque commande est complète et copiable : connexion VPN (`sudo openvpn ~/Downloads/FICHIER.ovpn`), vérification (`ip -4 addr show tun0`, `ping -c 2 IP_CIBLE`, `curl -sI http://IP_CIBLE:PORT/ | head -n 1`, `nmap`), connexion à la cible (`ssh`, `xfreerdp`, `evil-winrm`, `smbclient`, navigateur, etc. selon le module), jusqu'à la commande qui donne la réponse.
2. **Chaque question a sa propre procédure complète**, même si la connexion est la même qu'à la question précédente. Je dois pouvoir faire une question isolée sans relire les autres.
3. Pour un outil à interface (Kibana, Splunk, Wireshark, Burp…), décris les **clics** : quel menu, quel bouton, quel champ, quelle colonne ajouter, quel tri.
4. Indique le **résultat attendu** pour que je sache si ça marche (ex. : « ~68 hits », « Initialization Sequence Completed »).
5. Ajoute un **plan B** quand une commande peut échouer.
6. Pour les questions de pur cours (sans cible) : procédure en étapes avec la **section exacte du cours** où lire la phrase qui donne la réponse, et le raisonnement d'élimination des mauvaises options.
7. Une même question qui existe en plusieurs versions (options différentes) : une sous-section par version, plus une règle pour trancher.

### 🔌 Bloc de connexion (à adapter à chaque module)
Contient : Option A (Pwnbox) et Option B (VPN sur ma machine) avec les commandes, les tests de connexion, et les réglages de l'outil (ex. Kibana : data view, période, fuseau horaire). Adapte les ports et les outils au module (SMB 445, NFS 2049, WinRM 5985, SSH 22, RDP 3389, web 80/443, etc.).

### Style
- Français naturel, direct, sans blabla ni flatterie.
- Pas de tournures du type « ce n'est pas X, c'est Y » : affirme directement.
- Les termes techniques restent en anglais, avec la traduction entre parenthèses la première fois.
- Une table plutôt qu'un long paragraphe dès que le contenu est comparatif.

---

## ⚠️ RÈGLES D'HONNÊTETÉ

1. **Ne jamais inventer une réponse.** Si le cours ne la donne pas et que je ne te l'ai pas fournie, écris `*(à compléter)*`, explique la méthode et propose de compléter avec mes résultats.
2. Quand la méthode (requête, commande, champ) n'est pas dans le cours et que tu la **reconstitues**, signale-le avec un encadré `> ⚠️` et ton niveau de confiance, et donne un plan B.
3. Les réponses HTB restent **identiques** à celles du cours ou que je te donne (orthographe, majuscules, ponctuation, espaces).
4. Signale les incohérences ou erreurs repérées dans le cours.
5. N'invente jamais une sortie de commande, un nom de fichier ou une valeur : décris ce qu'on doit chercher.

---

## 📚 EXEMPLE DE RÉFÉRENCE (module déjà traité)

Voici la fiche complète du module « Introduction to Threat Hunting & Hunting With Elastic ». **Reproduis exactement cette structure, ce niveau de détail et ce style** pour le module que je t'enverrai. Le contenu de cet exemple sert uniquement de modèle de mise en forme. Tu ne réutilises pas ses informations pour un autre module.

---

# 🎯 Fiche de révision HTB CJCA — Introduction to Threat Hunting & Hunting With Elastic

> **Légende**
> - 📘 Résumé du cours (traduit en français)
> - 🔑 À retenir absolument
> - ⌨️ Requêtes / commandes importantes
> - ✅ Question, réponse en français, et **Réponse à mettre sur HTB** (en anglais, telle qu'attendue)
> - 🧭 Procédure détaillée, étape par étape, pour retrouver la réponse
> - ⚠️ Passage reconstitué (absent du cours). À vérifier sur la cible.

---

## Sommaire

1. [Page 1 — Threat Hunting Fundamentals](#page-1--threat-hunting-fundamentals)
2. [Page 2 — The Threat Hunting Process](#page-2--the-threat-hunting-process)
3. [Page 3 — Threat Hunting Glossary](#page-3--threat-hunting-glossary)
4. [Page 4 — Threat Intelligence Fundamentals](#page-4--threat-intelligence-fundamentals)
5. [Page 5 — Hunting For Stuxbot](#page-5--hunting-for-stuxbot)
6. [Page 6 — Skills Assessment : Hunting For Stuxbot (Round 2)](#page-6--skills-assessment--hunting-for-stuxbot-round-2)
7. [Mémo final : toutes les réponses HTB](#mémo-final--toutes-les-réponses-htb)
8. [Cheat-sheet KQL](#cheat-sheet-kql-du-module)

---

# 🔌 Bloc de connexion à la cible (valable pour les pages 5 et 6)

Les questions des pages 5 et 6 se font sur une machine cible HTB avec **Kibana** (port **5601**). Deux façons de s'y connecter.

## Option A — Depuis le Pwnbox (le plus simple)

| Étape | Action |
|---|---|
| A1 | En bas de la section, clique sur **Spawn Target** et note l'**IP cible** (ex. `10.129.xx.xx`) |
| A2 | Clique sur **Linux Pwnbox** → **View Linux Pwnbox** et attends le chargement du bureau |
| A3 | Ouvre **Firefox** dans le Pwnbox |
| A4 | Va sur `http://IP_CIBLE:5601` |
| A5 | Attends **3 à 5 minutes** après le spawn si la page ne répond pas, puis recharge (F5) |

Test en ligne de commande depuis un terminal du Pwnbox :
```bash
# Vérifie que Kibana répond (attendu : HTTP/1.1 200 OK ou 302 Found)
curl -sI http://IP_CIBLE:5601/ | head -n 1

# Vérifie que le port 5601 est ouvert (attendu : 5601/tcp open)
nmap -Pn -p 5601 IP_CIBLE
```

## Option B — Depuis ta propre machine (VPN)

| Étape | Commande / action |
|---|---|
| B1 | Section du cours → bouton **OVPN** → **View VPN** → **Download VPN Connection File** (fichier `.ovpn`) |
| B2 | Dans un terminal, lance le VPN (laisse ce terminal ouvert) : |

```bash
sudo openvpn ~/Downloads/NOM_DU_FICHIER.ovpn
# Attendu à la fin : "Initialization Sequence Completed"
```

| Étape | Commande / action |
|---|---|
| B3 | Dans un **second terminal**, vérifie l'interface VPN : |

```bash
ip -4 addr show tun0
# Attendu : une adresse du type 10.10.14.x
```

| Étape | Commande / action |
|---|---|
| B4 | Clique sur **Spawn Target**, note l'IP cible, puis teste la connexion : |

```bash
ping -c 2 IP_CIBLE
curl -sI http://IP_CIBLE:5601/ | head -n 1
nmap -Pn -p 5601 IP_CIBLE
```

| Étape | Commande / action |
|---|---|
| B5 | Ouvre ton navigateur sur `http://IP_CIBLE:5601` |

## Réglages Kibana à faire à chaque nouvelle cible

| Étape | Action | Résultat attendu |
|---|---|---|
| K1 | Menu latéral (☰ en haut à gauche) → **Discover** | La page de recherche s'ouvre |
| K2 | Icône **calendrier** (en haut à droite) → saisir **15** / **Years ago** → **Apply** (ou choisir « Last 15 years ») | Les logs de mars 2023 apparaissent |
| K3 | Ouvre `http://IP_CIBLE:5601/app/management/kibana/settings`, cherche **Timezone for date formatting** (`dateFormat:tz`), choisis **Europe/Copenhagen**, **Save changes** | Les horodatages correspondent à ceux du cours |
| K4 | En haut à gauche de Discover, menu déroulant des **data views** : choisis `windows*` (Sysmon, PowerShell, audit Windows) ou `zeek*` (réseau) | Le bon index est interrogé |
| K5 | Pour **ajouter une colonne** : survole un champ dans la liste de gauche → clique sur **+** (add) | Le champ s'affiche dans le tableau |
| K6 | Pour **trier** : survole l'en-tête de colonne (ex. `@timestamp`) → flèche de tri | Tri croissant ou décroissant |

---

# Page 1 — Threat Hunting Fundamentals

## 📘 Résumé

### Définition du Threat Hunting
- **Dwell time** (temps de séjour) = durée entre la compromission réelle et sa détection. Il se compte généralement en **semaines, voire en mois**.
- Les défenses classiques (réactives) ne suffisent plus. Le threat hunting ajoute une approche **proactive**.
- **Threat hunting** = pratique **active, menée par un humain**, souvent **basée sur des hypothèses**, qui fouille les données du réseau pour trouver des menaces furtives que les outils de sécurité existants ratent.
- **Objectif principal** : réduire le dwell time en repérant l'attaquant le plus tôt possible dans la **Cyber Kill Chain**.

### Déroulé général
1. Identifier les **assets** de grande valeur (systèmes, données).
2. Analyser les **TTPs** (Tactics, Techniques, Procedures) probables des attaquants grâce à la **Threat Intelligence**.
3. Détecter, isoler et valider les **artefacts** liés à ces TTPs et toute activité qui s'écarte de la **baseline** (comportement normal).

### Les 2 facettes du threat hunting
| Type | Description |
|---|---|
| **Proactif** | Anticipe les menaces à partir d'hypothèses, de TTPs et de renseignement |
| **Réactif** | Cherche dans le réseau les artefacts liés à un incident déjà vérifié, à partir de preuves et de renseignement |

### Ce qu'il faut pour bien chasser
- Comprendre le paysage des menaces, les TTPs adverses et la Kill Chain
- Se mettre à la place de l'attaquant (empathie cognitive)
- Connaître à fond son SI : topologie, assets, activité normale
- Utiliser des données de haute fidélité et des outils/plateformes adaptés

### Lien avec l'Incident Handling (gestion d'incident)
| Phase de l'incident handling | Rôle du threat hunter |
|---|---|
| **Préparation** | Définir des règles d'engagement claires (quand et comment intervenir). Peut être intégré aux procédures d'IR existantes |
| **Détection & Analyse** | Aide à confirmer si des IoCs correspondent à un vrai incident et trouve des IoCs manqués |
| **Confinement, Éradication, Récupération** | Rôle variable selon l'organisation. Défini dans les procédures |
| **Post-incident** | Recommandations pour renforcer la posture de sécurité |

🔑 Intégrer ou séparer le hunting et l'incident handling est une **décision stratégique** propre à chaque organisation, selon son paysage de menaces et ses risques.

### Structure d'une équipe de threat hunting
| Rôle | Mission |
|---|---|
| **Threat Hunter** | Rôle central, cherche proactivement les IoCs |
| **Threat Intelligence Analyst** | Collecte et analyse (OSINT, dark web, rapports, feeds) pour prédire les tendances |
| **Incident Responders** | Prennent le relais quand une menace est trouvée : investigation, confinement, éradication, récupération |
| **Forensics Experts** | DFIR : analyse de malware, reverse engineering, rapports détaillés |
| **Data Analysts / Scientists** | Statistiques, machine learning, data mining pour trouver des motifs |
| **Security Engineers / Architects** | Conçoivent l'infrastructure de sécurité et implémentent les outils de hunting |
| **Network Security Analyst** | Spécialiste du trafic réseau, repère les anomalies |
| **SOC Manager** | Supervise l'équipe et la coordination |

### Quand chasser ?
1. **Nouvelle information** sur un adversaire ou une vulnérabilité
2. **Nouveaux indicateurs** associés à un adversaire connu
3. **Plusieurs anomalies réseau** détectées en même temps
4. **Pendant une réponse à incident** (en parallèle de l'IR, pour trouver d'autres systèmes compromis)
5. **Périodiquement**, de façon proactive

🔑 « Le meilleur moment pour chasser, c'est toujours maintenant. »

### Lien avec le Risk Assessment (évaluation des risques)
Le risk assessment sert à :
- **Prioriser** la chasse (assets critiques = « crown jewels »)
- **Comprendre** le paysage de menaces et construire des hypothèses
- **Mettre en évidence** les vulnérabilités (ex. : faille d'escalade de privilèges, donc on cherche des anomalies de niveaux de privilège)
- **Orienter** l'usage de la threat intelligence
- **Affiner** les plans d'IR
- **Améliorer** les contrôles de sécurité

Outils cités : scanners de vulnérabilités, outils de pentest, plateformes de threat intelligence, **SIEM** (agrège et corrèle les événements).

## ✅ Questions de la page 1

### ❓ Question 1

| | |
|---|---|
| **Question (EN)** | Threat hunting is used ... Choose one: "proactively", "reactively", "proactively and reactively" |
| **Question (FR)** | Le threat hunting s'utilise ... (de façon proactive / réactive / proactive et réactive) |
| **Réponse (FR)** | De façon proactive **et** réactive |
| **Réponse à mettre sur HTB** | `proactively and reactively` |

**🧭 Procédure (aucune cible nécessaire, tout est dans le texte du cours)**

1. Ouvre la page **Threat Hunting Fundamentals** → section **Threat Hunting Definition**.
2. Descends jusqu'à la liste **« Key facets of threat hunting include »**.
3. Repère les deux premières puces :
   - « An offensive, **proactive** strategy… based on hypotheses, attacker TTPs, and intelligence »
   - « An offensive, **reactive** response… based on evidence and intelligence »
4. Deux facettes existent, donc la bonne option est la troisième.
5. Saisis exactement : `proactively and reactively`

✔️ **Vérification** : la page 4 confirme, avec « Threat Hunting (Reactive and Proactive) ».

---

### ❓ Question 2

| | |
|---|---|
| **Question (EN)** | Threat hunting and incident handling are two processes that always function independently. True/False |
| **Question (FR)** | Le threat hunting et l'incident handling fonctionnent toujours de façon indépendante. Vrai/Faux |
| **Réponse (FR)** | Faux |
| **Réponse à mettre sur HTB** | `false` |

**🧭 Procédure**

1. Repère le mot clé de l'énoncé : **« always »** (toujours). Une affirmation absolue se vérifie par un seul contre-exemple.
2. Va à la section **The Relationship Between Incident Handling & Threat Hunting**.
3. Lis la phrase de fin : intégrer ou séparer les deux processus est « a strategic decision, contingent upon each organization's unique threat landscape, risk ».
4. Lis aussi le début de la section : les organisations peuvent **intégrer** le hunting dans leurs procédures d'incident handling. Les deux ne sont donc pas toujours indépendants.
5. Saisis exactement : `false`

---

### ❓ Question 3

| | |
|---|---|
| **Question (EN)** | Threat hunting and incident response can be conducted simultaneously. True/False |
| **Question (FR)** | Le threat hunting et l'incident response peuvent être menés en même temps. Vrai/Faux |
| **Réponse (FR)** | Vrai |
| **Réponse à mettre sur HTB** | `true` |

**🧭 Procédure**

1. Va à la section **When Should We Hunt?**.
2. Repère la puce **« During an Incident Response Activity »**.
3. Lis : pendant que l'IR gère le confinement, l'éradication et la récupération, il faut **« simultaneously conduct threat hunting across the network »** pour trouver d'autres systèmes compromis.
4. Saisis exactement : `true`

---

# Page 2 — The Threat Hunting Process

## 📘 Résumé

Le processus se déroule en **7 étapes** (cycle continu) :

| # | Étape | Ce qu'on fait | Exemple |
|---|---|---|---|
| 1 | **Setting the Stage** (préparation) | Définir des objectifs, activer des **logs étendus**, configurer SIEM / EDR / IDS, se tenir informé des menaces | Lire des rapports CTI, identifier les assets critiques |
| 2 | **Formulating Hypotheses** (hypothèses) | Faire des prédictions **testables** à partir de CTI, d'alertes ou d'intuition | « Un groupe APT exploite une faille du serveur web pour établir un canal C2 » |
| 3 | **Designing the Hunt** (conception) | Choisir les sources de données, outils, IoCs, écrire des requêtes ou scripts | Logs web, DNS, télémétrie endpoint |
| 4 | **Data Gathering & Examination** (collecte et examen) | Collecter et analyser les données pour **confirmer ou réfuter** l'hypothèse. Étape itérative | Logs d'accès, captures réseau, logs endpoint |
| 5 | **Evaluating Findings & Testing Hypotheses** (évaluation) | Interpréter : hypothèse confirmée ou non, systèmes touchés, impact | Tentatives de brute force depuis une IP connue |
| 6 | **Mitigating Threats** (mitigation) | Isoler, supprimer le malware, patcher, modifier les configurations | Isoler une machine qui parle à un C2 |
| 7 | **After the Hunt** (après la chasse) | Documenter, partager, mettre à jour la threat intel, améliorer règles et playbooks | Nouveaux IoCs, règles de détection |
| ➕ | **Continuous Learning** | Chaque cycle nourrit le suivant | Nouvelles techniques (ML, analyse comportementale) |

🔑 **Mnémotechnique** : *Préparer → Hypothèse → Concevoir → Collecter → Évaluer → Mitiger → Documenter → (recommencer)*

### Exemple fil rouge : Emotet
| Étape | Application à Emotet |
|---|---|
| Préparation | Étudier les TTPs d'Emotet (pièces jointes/liens malveillants), cibler endpoints admin et serveurs mail |
| Hypothèse | « Emotet utilise des comptes mail compromis pour envoyer des documents Word avec macros » |
| Conception | Logs serveur mail, trafic réseau, logs endpoint, sandbox. Feeds CTI pour les IoCs (adresses C2, hashes) |
| Collecte | En-têtes d'emails, captures réseau, comportement |
| Évaluation | Emails avec objets similaires, connexions vers des C2 Emotet connus |
| Mitigation | Isoler les systèmes, supprimer le malware, bloquer les C2, patcher |
| Après la chasse | Mettre à jour les IoCs, règles de détection, playbooks |
| Apprentissage | Détection comportementale, ML |

## ✅ Question de la page 2

### ❓ Question 1

| | |
|---|---|
| **Question (EN)** | It is OK to formulate hypotheses that are not testable. True/False |
| **Question (FR)** | Il est acceptable de formuler des hypothèses non testables. Vrai/Faux |
| **Réponse (FR)** | Faux |
| **Réponse à mettre sur HTB** | `false` |

**🧭 Procédure**

1. Va à l'étape **Formulating Hypotheses** (2e étape de la liste).
2. Lis la phrase : « We strive to make these hypotheses **testable** to guide us where to search and what to look for ».
3. Lis l'exemple juste en dessous : « The hypothesis should be **specific and testable** ».
4. Une hypothèse non testable ne dit ni où chercher ni quoi chercher. L'affirmation de l'énoncé est donc fausse.
5. Saisis exactement : `false`

---

# Page 3 — Threat Hunting Glossary

> Cette page ne contient **aucune question**. Elle contient le vocabulaire utilisé dans les pages suivantes et à l'examen.

## 📘 Résumé : les définitions essentielles

| Terme | Définition en français |
|---|---|
| **Adversary** (adversaire) | Entité qui cherche à s'infiltrer dans l'organisation pour atteindre ses objectifs (gain financier, informations internes, propriété intellectuelle). Catégories : cybercriminels, menaces internes (insiders), hacktivistes, acteurs étatiques |
| **APT** (Advanced Persistent Threat) | Groupe très organisé ou étatique, avec beaucoup de ressources, actif sur de longues périodes. « Advanced » renvoie à la planification stratégique sophistiquée, sans exiger de technique avancée. « Persistent » renvoie à leur obstination |
| **TTPs** | Signature opérationnelle d'un adversaire |
| ↳ **Tactics** | Objectifs stratégiques : le **pourquoi** |
| ↳ **Techniques** | Méthodes générales : le **comment** |
| ↳ **Procedures** | Étapes détaillées, la « recette » |
| **Indicator** | **Données + contexte = indicateur**. Une donnée technique sans contexte a peu de valeur |
| **Threat** (menace) | **Intention + Capacité + Opportunité** |
| **Campaign** | Ensemble d'incidents partageant des TTPs similaires et des objectifs comparables |
| **IOCs** (Indicators of Compromise) | Traces numériques d'une intrusion : hashes, IPs, URLs, domaines, noms d'exécutables/scripts |

### La Pyramid of Pain (David Bianco)
Plus on monte, plus l'indicateur est **difficile à obtenir pour le défenseur**, et plus il est **coûteux à changer pour l'attaquant**.

```
            /\
           /TTPs\            ← Tough
          /------\
         / Tools  \          ← Challenging
        /----------\
       /Network/Host\        ← Annoying
      /  Artifacts   \
     /----------------\
    /   Domain Names   \     ← Simple
   /--------------------\
  /     IP Addresses     \   ← Easy
 /------------------------\
/       Hash Values        \ ← Trivial
```

| Niveau | Pourquoi |
|---|---|
| **Hash** | Un seul octet modifié change le hash. Trivial à changer, peu fiable |
| **IP** | VPN, proxy, TOR, spoofing. Facile à changer |
| **Domaine** | DGA (algorithmes de génération de domaines), DNS dynamique. Simple à changer |
| **Artefacts réseau/hôte** | Motifs de trafic, clés de registre, chemins de fichiers, processus. Gênant à changer sans casser l'opération |
| **Outils** | Malware, exploits, frameworks C2. Difficile à remplacer (les adversaires avancés les personnalisent) |
| **TTPs** | Le sommet : l'attaquant doit changer sa façon de travailler |

### Le Diamond Model (modèle du diamant)
| Sommet | Rôle |
|---|---|
| **Adversary** | Qui attaque |
| **Capability** | Outils, malware, exploits, TTPs |
| **Infrastructure** | Serveurs, domaines, IPs, botnets utilisés |
| **Victim** | Cible (personne, organisation, système) |

*Exemple du cours* : une institution financière (Victim) est ciblée par un groupe cybercriminel (Adversary) via du spear-phishing (Capability) envoyé depuis un botnet (Infrastructure) pour livrer un cheval de Troie bancaire.

🔑 La Cyber Kill Chain décrit les **étapes** d'une attaque. Le Diamond Model décrit les **composants** de l'intrusion et leurs relations. Les deux se complètent.

---

# Page 4 — Threat Intelligence Fundamentals

## 📘 Résumé

### Définition de la CTI (Cyber Threat Intelligence)
La CTI fait passer la défense d'un mode réactif à un mode **proactif et anticipatif**, et alimente le **SOC**.

### Les 4 principes d'une bonne CTI
| Principe | Signification |
|---|---|
| **Relevance** (pertinence) | L'info concerne **notre** organisation (une faille dans un logiciel qu'on n'utilise pas est moins urgente) |
| **Timeliness** (actualité) | La valeur baisse avec le temps. Les vieux indicateurs perdent en pertinence |
| **Actionability** (exploitabilité) | L'info doit donner des **actions claires** à l'équipe de défense. Sinon : « self-licking ice cream cone » (analyse stérile qui tourne en rond) |
| **Accuracy** (exactitude) | Vérifier avant de diffuser. Si incertain, ajouter un **indicateur de confiance** |

### Ce que la CTI apporte
- Comprendre les opérations et campagnes adverses
- Enrichir les données par l'analyse
- Découvrir les TTPs pour construire des mesures de mitigation
- Aider les décideurs à décider

### Threat Intelligence vs Threat Hunting
| | **Threat Intelligence** | **Threat Hunting** |
|---|---|---|
| Nature | **Prédictive** | **Réactive et proactive** |
| But | Anticiper où, quand, comment l'adversaire attaquera et ses objectifs | Vérifier si un adversaire est présent (ou l'a été sans être détecté) après un événement déclencheur |

Les deux se renforcent : la CTI informe le hunting, et les résultats du hunting enrichissent la CTI.

### Les 3 niveaux de renseignement
| Niveau | Public | Contenu | Répond à |
|---|---|---|---|
| **Strategic** | Dirigeants (C-suite, VPs) | Vue d'ensemble dans le temps, TTPs/MO, alignement avec les risques | **Qui ? Pourquoi ?** |
| **Operational** | Management intermédiaire | Détail des campagnes, TTPs | **Comment ? Où ?** |
| **Tactical** | Défenseurs réseau / SOC | Actions immédiates, détails techniques (IPs, domaines, hashes, clés de registre, mutex) | Quoi bloquer/détecter maintenant |

*Exemples du cours* : APT28 (stratégique), campagne ransomware REvil (opérationnel), IoCs REvil (tactique).

### Comment traiter un rapport de CTI tactique (6 étapes)
1. **Comprendre** la portée et le récit du rapport (la menace nous concerne-t-elle ?)
2. **Repérer et classer** les IoCs : réseau (IPs, domaines), hôte (hashes, clés de registre), email (adresses, objets), + mutex, certificats SSL, User-Agents
3. **Comprendre le cycle de vie** de l'attaque (TTPs mappés sur **MITRE ATT&CK**)
4. **Analyser et valider** les IoCs : VirusTotal, AlienVault OTX, âge de l'IoC, contexte (une IP C2 peut aussi héberger des sites légitimes), taux de faux positifs
5. **Intégrer** dans l'infrastructure : règles pare-feu, EDR, IDS/IPS, passerelle mail. Parfois **alerter plutôt que bloquer** pour ne pas casser un service critique. Documenter via le change management
6. **Chasser** de façon proactive (IoCs + TTPs, ex. PowerShell suspect), **surveiller** en continu et **partager** (ISACs/ISAOs)

## ✅ Questions de la page 4

### ❓ Question 1

| | |
|---|---|
| **Question (EN)** | It's useful for the CTI team to provide a single IP with no context to the SOC team. True/False |
| **Question (FR)** | Il est utile que la CTI donne au SOC une seule IP sans contexte. Vrai/Faux |
| **Réponse (FR)** | Faux |
| **Réponse à mettre sur HTB** | `false` |

**🧭 Procédure**

1. Va à la page **Threat Hunting Glossary**, définition **Indicator**.
2. Lis : « Isolated technical data lacking relevant context holds limited or negligible value for network defenders ».
3. Retiens la formule : **Data + context = indicator**. Une IP seule n'a pas de contexte.
4. Recoupe avec le principe d'**Actionability** (page 4) : l'info doit donner des directives claires.
5. Saisis exactement : `false`

---

### ❓ Question 2

| | |
|---|---|
| **Question (EN)** | When an incident occurs on the network and the CTI team is made aware, what should they do? Options : "Do Nothing", "Reach out to the Incident Handler/Incident Responder", "Provide IOCs on all research being conducted, regardless if the IOC is verified" |
| **Réponse (FR)** | Contacter l'Incident Handler / Incident Responder |
| **Réponse à mettre sur HTB** | `Reach out to the Incident Handler/Incident Responder` |

**🧭 Procédure**

1. Élimine d'abord les mauvaises options :
   - `Do Nothing` : contraire à l'objectif d'une CTI proactive.
   - `Provide IOCs ... regardless if the IOC is verified` : contraire au principe d'**Accuracy** (vérifier avant de diffuser).
2. Il reste `Reach out to the Incident Handler/Incident Responder`. Le cours (section *Difference Between Threat Intelligence & Threat Hunting*) indique que la CTI et les équipes opérationnelles échangent leurs informations.
3. Saisis exactement : `Reach out to the Incident Handler/Incident Responder`

---

### ❓ Question 3 (même question, autres options)

| | |
|---|---|
| **Question (EN)** | When an incident occurs on the network and the CTI team is made aware, what should they do? Options : "Provide IOCs on all research being conducted, regardless if the IOC is verified", "Do Nothing", "Provide further IOCs and TTPs associated with the incident" |
| **Réponse (FR)** | Fournir d'autres IoCs et TTPs associés à l'incident |
| **Réponse à mettre sur HTB** | `Provide further IOCs and TTPs associated with the incident` |

**🧭 Procédure**

1. Lis bien les options : la question est identique, la liste change.
2. Élimine `Do Nothing` et `Provide IOCs ... regardless if the IOC is verified` (principe d'**Accuracy**).
3. Il reste `Provide further IOCs and TTPs associated with the incident`, qui respecte les 4 critères (pertinent, actuel, actionnable, exact).
4. Saisis exactement : `Provide further IOCs and TTPs associated with the incident`

🔑 **Règle pour les deux versions** : si l'option « Reach out to the Incident Handler/Incident Responder » est proposée, prends-la. Sinon, prends « Provide further IOCs and TTPs… ».

---

### ❓ Question 4

| | |
|---|---|
| **Question (EN)** | Cyber Threat Intelligence, if curated and analyzed properly, can ... ? Options : "be used for security awareness", "be used for fine-tuning network segmentation", "provide insight into adversary operations" |
| **Réponse (FR)** | Donner un aperçu des opérations de l'adversaire |
| **Réponse à mettre sur HTB** | `provide insight into adversary operations` |

**🧭 Procédure**

1. Va à la section **Criteria Of Cyber Threat Intelligence**.
2. Lis : les 4 éléments (Actionable, Timely, Relevant, Accurate) forment la base d'une CTI robuste « that ultimately provides **visibility into adversary operations** ».
3. Les deux autres options ne figurent pas dans le cours comme apports de la CTI.
4. Saisis exactement : `provide insight into adversary operations`

---

# Page 5 — Hunting For Stuxbot

## 📘 Résumé

### Le rapport de CTI sur Stuxbot
- Groupe cybercriminel organisé, phishing **opportuniste** (« anyone, anytime »), motivation apparente : **espionnage**.
- **Plateforme ciblée** : Windows. **Niveau de risque** : Critique.
- **Chaîne d'attaque** : email de phishing → fichier **OneNote** (`invoice.one`) hébergé sur un service de partage → **fichier batch** (`invoice.bat`) → **script PowerShell** (stage 0, en mémoire) → **RAT** (exécutable pour la persistance).
- **RAT modulaire** : capture d'écran, Mimikatz, shell CMD interactif.
- **Persistance** : un EXE déposé sur le disque.
- **Mouvement latéral** : **PsExec** (signé Microsoft) et **WinRM**.

### IoCs du rapport
| Type | Valeurs |
|---|---|
| Fichiers OneNote | `transfer.sh/get/kNxU7/invoice.one`, `mega.io/dl9o1Dz/invoice.one` |
| Scripts PowerShell | `pastebin.com/raw/AvHtdKb2`, `pastebin.com/raw/gj58DKz` |
| Nœuds C2 | `91.90.213.14:443`, `103.248.70.64:443`, `141.98.6.59:443` |
| Hashes SHA256 | `226A723F…`, `C346077D…`, `018D37CB…` |

### L'environnement
- ~200 employés (marketing en ligne), Gmail dans le navigateur, **Microsoft Edge** par défaut, TeamViewer, GPO via Active Directory.
- **Logs** (Elastic Stack comme SIEM) :
  - `windows*` : logs d'audit Windows + **Sysmon** + logs **PowerShell** (~118 975 logs)
  - `zeek*` : logs réseau **Zeek** (~332 261 logs)
- Les données CTI datent de **mars 2023** : période à régler sur **« last 15 years »**.

### ⌨️ Les événements à connaître
| Event ID | Nom | Utilité |
|---|---|---|
| **1** | Sysmon : Process creation | Processus lancés, arguments, parent |
| **3** | Sysmon : Network connection | Connexions réseau (navigateurs souvent exclus de la config) |
| **11** | Sysmon : File create | Création de fichiers (`Zone.Identifier` = fichier venant d'Internet) |
| **13** | Sysmon : Registry value set | Modifications de registre (persistance Run keys) |
| **15** | Sysmon : FileCreateStreamHash | Téléchargement depuis un navigateur |
| **22** | Sysmon : DNSEvent | Requêtes DNS |
| **4624 / 4625** | Windows Security : logon réussi / échoué | Brute force, mouvement latéral |
| **4104** | PowerShell : Script Block Logging | Contenu des scripts PowerShell exécutés |

### 🔎 Déroulé de la chasse du cours (hypothèse : phishing réussi avec un OneNote malveillant)

| Étape | Requête KQL | Ce qu'on découvre |
|---|---|---|
| 1. Téléchargement du fichier | `event.code:15 AND file.name:*invoice.one` | 3 résultats. Téléchargé par **MSEdge** dans le dossier Downloads de **Bob**. Horodatage : **26 mars 2023 @ 22:05:47** |
| 2. Confirmation | `event.code:11 AND file.name:invoice.one*` | Machine **WS001**, avec un `Zone.Identifier` (le `*` est à la fin car le nom contient `:Zone.Identifier`). IP **192.168.28.130** |
| 3. IP de WS001 | `event.code:3 AND host.hostname:WS001` (champ `source.ip`) | Confirme l'IP |
| 4. DNS Zeek (22:05:00 à 22:05:48) | `source.ip:192.168.28.130 AND dns.question.name:*` | Filtrer le bruit (google.com, etc.). On voit **mail.google.com**, puis **file.io**, puis SmartScreen |
| 5. IPs de file.io | Champ `dns.answers.data` | `34.197.10.85`, `3.213.216.16` |
| 6. Connexions vers ces IPs | Recherche de `destination.ip` sur ces IPs | Port 443 : **Bob a téléchargé `invoice.one` depuis file.io** |
| 7. Ouverture du fichier | `event.code:1 AND process.command_line:*invoice.one*` | OneNote lance le fichier ~**6 secondes** après le téléchargement |
| 8. Enfants de OneNote | `event.code:1 AND process.parent.name:"ONENOTE.EXE"` | `OneNoteM.exe` (normal) et **`cmd.exe` qui exécute `invoice.bat`** |
| 9. Enfants du batch | `event.code:1 AND process.parent.command_line:*invoice.bat*` | **PowerShell** qui télécharge un script **Pastebin** (PID **9944**) |
| 10. Activité PowerShell | `process.pid:"9944" and process.name:"powershell.exe"` | 17 événements : script de brute force, dépôt d'un EXE, DNS **ngrok**, connexions vers le C2 (`18.158.249.75`), requêtes vers **DC1** |
| 11. Zeek sur l'IP C2 | `destination.ip:18.158.249.75` | Activité qui continue le lendemain. L'IP de ngrok change ensuite (`3.125.102.39`) |
| 12. Le dropper | `process.name:"default.exe"` | Exécuté. Dépose **`svchost.exe`**, **`SharpHound.exe`**, `payload.exe`, un **fichier VBS** |
| 13. SharpHound | `process.name:"SharpHound.exe"` | Exécuté **2 fois** (~2 minutes d'écart), collecte `all` : cartographie de l'Active Directory |
| 14. Hash connu | `process.hash.sha256:018d37cb…` | Hash présent sur **WS001** et **PKI** |
| 15. Mouvement latéral | Parent = **PSEXESVC** sur PKI | **PsExec**. Utilisateur compromis : **svc-sql1** |
| 16. Brute force | `(event.code:4624 OR event.code:4625) AND winlog.event_data.LogonType:3 AND source.ip:192.168.28.130` | 2 échecs sur **administrator**, puis des connexions réussies de **svc-sql1** (le 28 mars) |

🔑 **Points d'analyse**
- Le rapport CTI cite `mega.io` et `transfer.sh`, alors que l'incident réel passe par **`file.io`**. Les IoCs d'un rapport sont incomplets, et la chasse par TTPs trouve ce que la chasse par IoCs rate.
- Les connexions réseau des navigateurs sont exclues de la config Sysmon. Les **logs Zeek** comblent ce manque.
- Les délais entre certaines actions suggèrent une **intervention humaine**.

## ✅ Questions de la page 5

> ⚠️ Le cours ne détaille pas les requêtes exactes pour ces 3 questions. Les **réponses** sont validées. Les **requêtes** sont reconstituées à partir de la logique du cours (confiance : bonne pour Q1 et Q2, moyenne pour Q3). Un plan B est donné à chaque fois.

### ❓ Question 1 — Le fichier VBS

| | |
|---|---|
| **Question (EN)** | In the part where default.exe is under investigation, a VBS file is mentioned. Enter its full name, including the extension. |
| **Question (FR)** | Dans la partie sur `default.exe`, un fichier VBS est mentionné. Donne son nom complet avec l'extension. |
| **Réponse (FR)** | `XceGuhkzaTrOy.vbs` |
| **Réponse à mettre sur HTB** | `XceGuhkzaTrOy.vbs` |

**🧭 Procédure complète**

**Partie 0 : connexion à la cible**

1. Sur la page, clique sur **Spawn Target** et note l'IP (`IP_CIBLE`).
2. Connexion, au choix :
   - **Pwnbox** : clique sur *View Linux Pwnbox*, ouvre Firefox.
   - **VPN** :
     ```bash
     sudo openvpn ~/Downloads/NOM_DU_FICHIER.ovpn     # terminal 1, laisser ouvert
     ip -4 addr show tun0                              # terminal 2 : doit afficher une IP 10.10.14.x
     ping -c 2 IP_CIBLE
     ```
3. Teste Kibana (attendre 3 à 5 minutes après le spawn) :
   ```bash
   curl -sI http://IP_CIBLE:5601/ | head -n 1
   ```
4. Ouvre `http://IP_CIBLE:5601` dans le navigateur.
5. Règle le fuseau : `http://IP_CIBLE:5601/app/management/kibana/settings` → **Europe/Copenhagen** → **Save changes**.

**Partie 1 : la recherche**

6. Menu ☰ → **Discover**. Data view : `windows*`.
7. Calendrier → **15 Years ago** → **Apply**.
8. Saisis dans la barre de recherche :
   ```
   process.name:"default.exe"
   ```
9. Ajoute les colonnes (survol du champ à gauche → **+**) : `process.name`, `event.code`, `file.path`, `destination.ip`, `dns.question.name`. Tu dois voir ~**68 hits**.
10. Fais défiler le tableau vers le haut et le bas. Le cours indique qu'on y voit le dépôt de `payload.exe`, `svchost.exe`, `SharpHound.exe` et **un fichier VBS**.
11. Lis la colonne `file.path` : repère la ligne qui se termine par `.vbs`. Le nom est `XceGuhkzaTrOy.vbs`.

✔️ **Plan B** (recherche directe) :
```
event.code:11 AND file.name:*.vbs
```
Puis vérifie que `process.name` vaut `default.exe`.

**Partie 2 : validation**

12. Retourne sur la page HTB, saisis `XceGuhkzaTrOy.vbs` (respecte les majuscules et minuscules) et clique sur **Submit**.

---

### ❓ Question 2 — Arguments de Mimikatz

| | |
|---|---|
| **Question (EN)** | Stuxbot uploaded and executed mimikatz. Provide the process arguments (what is after .\mimikatz.exe, ...) as your answer. |
| **Question (FR)** | Stuxbot a téléversé et exécuté mimikatz. Donne les arguments du processus (ce qui suit `.\mimikatz.exe`). |
| **Réponse (FR)** | Commande DCSync sur le domaine eagle.local |
| **Réponse à mettre sur HTB** | `lsadump::dcsync /domain:eagle.local /all /csv, exit` |

**🧭 Procédure complète**

**Partie 0 : connexion à la cible** (identique à la Question 1, étapes 1 à 7)

1. **Spawn Target**, note `IP_CIBLE`.
2. Connexion Pwnbox, ou VPN :
   ```bash
   sudo openvpn ~/Downloads/NOM_DU_FICHIER.ovpn
   ip -4 addr show tun0
   ping -c 2 IP_CIBLE
   curl -sI http://IP_CIBLE:5601/ | head -n 1
   ```
3. Ouvre `http://IP_CIBLE:5601` → **Discover** → data view `windows*` → **15 Years ago** → **Apply**.
4. Fuseau **Europe/Copenhagen** (`/app/management/kibana/settings`).

**Partie 1 : la recherche**

5. Requête :
   ```
   process.name:"mimikatz.exe"
   ```
6. Ajoute les colonnes : `process.name`, `process.args`, `process.command_line`, `host.hostname`, `user.name`.
7. Ouvre un document (flèche à gauche de la ligne) pour voir tous les champs.
8. Lis `process.args` : les valeurs apparaissent après `mimikatz.exe`. Recopie-les **exactement**, dans l'ordre, en gardant la virgule avant `exit` :
   ```
   lsadump::dcsync /domain:eagle.local /all /csv, exit
   ```

✔️ **Plan B** si `process.name` ne renvoie rien :
```
process.command_line:*mimikatz*
```
ou `event.code:1 AND process.command_line:*dcsync*`.

**Partie 2 : comprendre la commande**

| Élément | Signification |
|---|---|
| `lsadump::dcsync` | Module Mimikatz qui imite un contrôleur de domaine pour demander la réplication des hashes de mots de passe (MITRE T1003.006) |
| `/domain:eagle.local` | Domaine ciblé |
| `/all` | Tous les comptes |
| `/csv` | Sortie au format CSV |
| `exit` | Quitte Mimikatz |

**Partie 3 : validation**

9. Saisis la réponse exactement comme ci-dessus (espaces et virgule compris) → **Submit**.

---

### ❓ Question 3 — L'outil PowerShell chargé en mémoire

| | |
|---|---|
| **Question (EN)** | Some PowerShell code has been loaded into memory that scans/targets network shares. Leverage the available PowerShell logs to identify from which popular hacking tool this code derives. Answer format (one word): P____V___ |
| **Question (FR)** | Du code PowerShell chargé en mémoire scanne les partages réseau. Avec les logs PowerShell, trouve de quel outil de hacking connu il provient (un mot : `P____V___`). |
| **Réponse (FR)** | PowerView |
| **Réponse à mettre sur HTB** | `PowerView` |

**🧭 Procédure complète**

**Partie 0 : connexion à la cible** (identique aux questions précédentes)

1. **Spawn Target** → `IP_CIBLE`.
2. Pwnbox, ou VPN :
   ```bash
   sudo openvpn ~/Downloads/NOM_DU_FICHIER.ovpn
   ip -4 addr show tun0
   curl -sI http://IP_CIBLE:5601/ | head -n 1
   ```
3. `http://IP_CIBLE:5601` → **Discover** → data view `windows*` → **15 Years ago** → **Apply**.

**Partie 1 : la recherche** ⚠️ *méthode reconstituée, confiance moyenne*

4. Les logs PowerShell sont dans l'index `windows*`. Le **Script Block Logging** enregistre le contenu des scripts, même chargés en mémoire, sous l'event ID **4104**.
5. Requête de départ :
   ```
   event.code:4104
   ```
6. Ajoute les colonnes `host.hostname`, `user.name`, et le champ qui contient le code du script (selon l'index : `powershell.file.script_block_text` ou `message`).
7. Affine avec des mots typiques d'un scan de partages :
   ```
   event.code:4104 AND "Invoke-ShareFinder"
   ```
   puis, si rien ne sort : `event.code:4104 AND "Find-DomainShare"`.
8. Ouvre le document : les noms de fonctions du script (`Get-Net…`, `Invoke-ShareFinder`, `Find-DomainShare`, …) et les commentaires d'en-tête appartiennent à **PowerView**, un script de reconnaissance Active Directory de la suite **PowerSploit**.
9. Contrôle du format : `P____V___` = P + 4 lettres + V + 3 lettres → **PowerView** (P-o-w-e-r / V-i-e-w).

✔️ **Plan B** : recherche en texte libre sur le mot `PowerView` ou `PowerSploit` dans `windows*`.

**Partie 2 : validation**

10. Saisis `PowerView` (P et V en majuscules) → **Submit**.

---

# Page 6 — Skills Assessment : Hunting For Stuxbot (Round 2)

## 📘 Résumé

### Nouvelles informations sur la dernière version de Stuxbot
1. Utilise le dossier **`C:\Users\Public`** pour déposer des outils supplémentaires (**Lateral Tool Transfer**, MITRE T1570).
2. Utilise les **clés de registre Run** pour la persistance (MITRE **T1547.001**).
3. Utilise **PowerShell Remoting** (WinRM) pour le mouvement latéral et pour atteindre les contrôleurs de domaine.

### Environnement
Index `windows*` (audit Windows, Sysmon, PowerShell) et `zeek*`. Même préparation qu'à la page 5 (voir le **Bloc de connexion** en haut de la fiche).

## ✅ Les 3 hunts

> ⚠️ **Réponses validées sur HTB.** Les **requêtes** viennent d'une reconstitution à partir de la logique du cours (confiance : bonne pour Hunt 2, moyenne pour Hunt 1 et Hunt 3). Un plan B accompagne chaque hunt. Si un nom de champ diffère sur ta cible, bascule sur le plan B.

### 🔎 Hunt 1 — Lateral Tool Transfer vers `C:\Users\Public`

| | |
|---|---|
| **Énoncé (EN)** | Create a KQL query to hunt for "Lateral Tool Transfer" to `C:\Users\Public`. Enter the content of the `user.name` field in the document that is related to a transferred tool that starts with "r". |
| **Énoncé (FR)** | Crée une requête KQL pour chasser un « Lateral Tool Transfer » vers `C:\Users\Public`. Donne le contenu du champ `user.name` du document lié à un outil transféré dont le nom commence par « r ». |
| **Réponse (FR)** | Le compte compromis `svc-sql1` |
| **Réponse à mettre sur HTB** | `svc-sql1` |

**🧭 Procédure complète**

**Partie 0 : connexion à la cible**

1. **Spawn Target** (en bas de la page) → note `IP_CIBLE`.
2. Connexion :
   - **Pwnbox** : *View Linux Pwnbox* → Firefox.
   - **VPN** :
     ```bash
     sudo openvpn ~/Downloads/NOM_DU_FICHIER.ovpn     # terminal 1
     ip -4 addr show tun0                              # terminal 2
     ping -c 2 IP_CIBLE
     ```
3. Test :
   ```bash
   curl -sI http://IP_CIBLE:5601/ | head -n 1
   ```
   (patiente 3 à 5 minutes si pas de réponse)
4. Navigateur : `http://IP_CIBLE:5601`
5. Fuseau : `http://IP_CIBLE:5601/app/management/kibana/settings` → **Europe/Copenhagen** → **Save changes**.

**Partie 1 : la chasse**

6. ☰ → **Discover**, data view `windows*`, calendrier → **15 Years ago** → **Apply**.
7. Requête (fichiers créés dans le dossier Public) :
   ```
   event.code:11 AND file.path:*Users\\Public*
   ```
   Syntaxe : dans KQL, chaque `\` d'un chemin Windows s'écrit `\\`.
8. Ajoute les colonnes : `file.name`, `file.path`, `user.name`, `host.hostname`, `process.name`.
9. Cible le fichier qui commence par « r » :
   ```
   event.code:11 AND file.path:*Users\\Public* AND file.name:r*
   ```
10. Ouvre le document trouvé et lis `user.name` : `svc-sql1`.

✔️ **Cohérence avec la page 5** : `svc-sql1` est le compte compromis par brute force, utilisé pour le mouvement latéral (PsExec). Un outil copié sur une autre machine avec ce compte confirme le transfert latéral.

✔️ **Plan B** :
- Sans filtre sur `event.code` : `file.path:*Public* AND file.name:r*`
- Via le réseau (SMB) dans `zeek*` : `destination.port:445`, puis recouper les horodatages.

**Partie 2 : validation**

11. Saisis `svc-sql1` → **Submit** (champ « Hunt 1 »).

---

### 🔎 Hunt 2 — Persistance par clés Run du registre

| | |
|---|---|
| **Énoncé (EN)** | Create a KQL query to hunt for "Boot or Logon Autostart Execution: Registry Run Keys / Startup Folder". Enter the content of the `registry.value` field in the document that is related to the first registry-based persistence action. |
| **Énoncé (FR)** | Crée une requête KQL pour chasser la persistance par clés Run / dossier Démarrage. Donne le contenu du champ `registry.value` du document lié à la **première** action de persistance par registre. |
| **Réponse (FR)** | Valeur de registre créée |
| **Réponse à mettre sur HTB** | `LgvHsviAUVTsIN` |

**🧭 Procédure complète**

**Partie 0 : connexion à la cible** (mêmes étapes que le Hunt 1)

1. **Spawn Target** → `IP_CIBLE`.
2. Pwnbox, ou VPN :
   ```bash
   sudo openvpn ~/Downloads/NOM_DU_FICHIER.ovpn
   ip -4 addr show tun0
   ping -c 2 IP_CIBLE
   curl -sI http://IP_CIBLE:5601/ | head -n 1
   ```
3. `http://IP_CIBLE:5601` → fuseau **Europe/Copenhagen**.

**Partie 1 : la chasse**

4. ☰ → **Discover**, data view `windows*`, **15 Years ago** → **Apply**.
5. Requête (Sysmon event 13 = valeur de registre écrite, sur la clé Run) :
   ```
   event.code:13 AND registry.path:*CurrentVersion\\Run*
   ```
6. Ajoute les colonnes : `@timestamp`, `registry.path`, `registry.value`, `process.name`, `host.hostname`, `user.name`.
7. **Trie par date croissante** (survole l'en-tête `@timestamp` → flèche de tri) pour avoir l'action la **plus ancienne** en premier.
8. Ouvre la première ligne pertinente (écriture dans une clé `…\Run\…`) et lis `registry.value` : `LgvHsviAUVTsIN`.

✔️ **Plan B** :
```
registry.path:*Run*
```
ou sans `event.code`. Les majuscules et minuscules comptent dans la réponse.

**Partie 2 : validation**

9. Saisis `LgvHsviAUVTsIN` → **Submit** (champ « Hunt 2 »).

---

### 🔎 Hunt 3 — PowerShell Remoting vers DC1

| | |
|---|---|
| **Énoncé (EN)** | Create a KQL query to hunt for "PowerShell Remoting for Lateral Movement". Enter the content of the `winlog.user.name` field in the document that is related to PowerShell remoting-based lateral movement towards DC1. |
| **Énoncé (FR)** | Crée une requête KQL pour chasser le mouvement latéral par PowerShell Remoting. Donne le contenu du champ `winlog.user.name` du document lié au PowerShell Remoting vers DC1. |
| **Réponse (FR)** | Le compte compromis `svc-sql1` |
| **Réponse à mettre sur HTB** | `svc-sql1` |

**🧭 Procédure complète**

**Partie 0 : connexion à la cible** (mêmes étapes que le Hunt 1)

1. **Spawn Target** → `IP_CIBLE`.
2. Pwnbox, ou VPN :
   ```bash
   sudo openvpn ~/Downloads/NOM_DU_FICHIER.ovpn
   ip -4 addr show tun0
   ping -c 2 IP_CIBLE
   curl -sI http://IP_CIBLE:5601/ | head -n 1
   ```
3. `http://IP_CIBLE:5601` → fuseau **Europe/Copenhagen**.

**Partie 1 : la chasse**

4. ☰ → **Discover**, data view `windows*`, **15 Years ago** → **Apply**.
5. Principe : un PowerShell Remoting entrant démarre le processus **`wsmprovhost.exe`** sur la machine cible, et utilise WinRM (ports **5985** HTTP / **5986** HTTPS). Requête de départ :
   ```
   event.code:1 AND process.name:"wsmprovhost.exe"
   ```
6. Ajoute les colonnes : `@timestamp`, `host.hostname`, `process.name`, `process.parent.name`, `user.name`, `winlog.user.name`.
7. Garde les lignes où `host.hostname` correspond à **DC1** (ajoute `AND host.hostname:DC1` si besoin).
8. Lis `winlog.user.name` dans le document : `svc-sql1`.

✔️ **Plan B** (si `winlog.user.name` est vide dans le document Sysmon) : ce champ vient de l'en-tête des événements Windows (Security, PowerShell), pas toujours de Sysmon. Dans ce cas :
```
host.hostname:DC1 AND event.code:4624 AND winlog.event_data.LogonType:3
```
ou
```
host.hostname:DC1 AND process.name:"wsmprovhost.exe"
```
puis ouvre le document et lis `winlog.user.name`. Côté réseau (`zeek*`) : `destination.port:(5985 OR 5986)` pour identifier la machine source du remoting.

**Partie 2 : validation**

9. Saisis `svc-sql1` → **Submit** (champ « Hunt 3 »).

---

# Mémo final — toutes les réponses HTB

| Page | Question | **Réponse à mettre sur HTB** |
|---|---|---|
| 1 | Threat hunting utilisé... | `proactively and reactively` |
| 1 | Hunting et incident handling toujours indépendants ? | `false` |
| 1 | Hunting et IR simultanés ? | `true` |
| 2 | Hypothèses non testables OK ? | `false` |
| 3 | *(pas de question)* | — |
| 4 | Une seule IP sans contexte utile ? | `false` |
| 4 | Incident + CTI prévenue (version 1) | `Reach out to the Incident Handler/Incident Responder` |
| 4 | Incident + CTI prévenue (version 2) | `Provide further IOCs and TTPs associated with the incident` |
| 4 | CTI bien analysée peut... | `provide insight into adversary operations` |
| 5 | Fichier VBS de default.exe | `XceGuhkzaTrOy.vbs` |
| 5 | Arguments de mimikatz | `lsadump::dcsync /domain:eagle.local /all /csv, exit` |
| 5 | Outil derrière le code PowerShell | `PowerView` |
| 6 | Hunt 1 (`user.name`) | `svc-sql1` |
| 6 | Hunt 2 (`registry.value`) | `LgvHsviAUVTsIN` |
| 6 | Hunt 3 (`winlog.user.name`) | `svc-sql1` |

---

# Cheat-sheet KQL du module

| Objectif | Requête |
|---|---|
| Téléchargement navigateur d'un fichier | `event.code:15 AND file.name:*invoice.one` |
| Création de fichier (avec Zone.Identifier) | `event.code:11 AND file.name:invoice.one*` |
| Connexions réseau d'un hôte | `event.code:3 AND host.hostname:WS001` |
| DNS Zeek d'une machine | `source.ip:192.168.28.130 AND dns.question.name:*` |
| Process lancé avec un fichier en argument | `event.code:1 AND process.command_line:*invoice.one*` |
| Enfants d'un processus (par nom du parent) | `event.code:1 AND process.parent.name:"ONENOTE.EXE"` |
| Enfants d'un processus (par ligne de commande du parent) | `event.code:1 AND process.parent.command_line:*invoice.bat*` |
| Activité d'un processus précis | `process.pid:"9944" and process.name:"powershell.exe"` |
| Exécution d'un binaire | `process.name:"default.exe"` |
| Recherche par hash | `process.hash.sha256:<hash en minuscules>` |
| Logons réseau réussis/échoués depuis une IP | `(event.code:4624 OR event.code:4625) AND winlog.event_data.LogonType:3 AND source.ip:192.168.28.130` |
| Fichiers dans le dossier Public | `event.code:11 AND file.path:*Users\\Public*` |
| Persistance par clé Run | `event.code:13 AND registry.path:*CurrentVersion\\Run*` |
| PowerShell Remoting (cible) | `event.code:1 AND process.name:"wsmprovhost.exe"` |
| Scripts PowerShell (contenu) | `event.code:4104` |

**Syntaxe KQL** : `AND` / `OR` / `NOT`, `*` pour joker, guillemets pour une valeur exacte, `\\` pour chaque antislash d'un chemin Windows, parenthèses pour grouper (`destination.port:(5985 OR 5986)`).