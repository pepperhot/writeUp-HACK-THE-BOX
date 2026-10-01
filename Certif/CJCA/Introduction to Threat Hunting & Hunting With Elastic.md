# 🎯 Fiche de révision HTB CJCA — Introduction to Threat Hunting & Hunting With Elastic

> **Légende**
> - 📘 Résumé du cours (traduit en français)
> - 🔑 À retenir absolument
> - ⌨️ Requêtes / commandes importantes
> - ✅ Question + réponse en français + **réponse à mettre sur HTB**
> - 🧭 Comment arriver à la réponse

---

## Sommaire

1. [Page 1 — Threat Hunting Fundamentals](#page-1--threat-hunting-fundamentals)
2. [Page 2 — The Threat Hunting Process](#page-2--the-threat-hunting-process)
3. [Page 3 — Threat Hunting Glossary](#page-3--threat-hunting-glossary)
4. [Page 4 — Threat Intelligence Fundamentals](#page-4--threat-intelligence-fundamentals)
5. [Page 5 — Hunting For Stuxbot](#page-5--hunting-for-stuxbot)
6. [Page 6 — Skills Assessment : Hunting For Stuxbot (Round 2)](#page-6--skills-assessment--hunting-for-stuxbot-round-2)
7. [Mémo final : tableau de toutes les réponses](#mémo-final--toutes-les-réponses-htb)

---

# Page 1 — Threat Hunting Fundamentals

## 📘 Résumé

### Définition du Threat Hunting
- **Dwell time** (temps de séjour) = durée entre la compromission réelle et sa détection. Il se compte généralement en **semaines, voire en mois**.
- Les défenses classiques (réactives) ne suffisent plus. Le threat hunting ajoute une approche **proactive**.
- **Threat hunting** = pratique **active, menée par un humain**, souvent **basée sur des hypothèses**, qui fouille les données du réseau pour trouver des menaces furtives qui échappent aux outils de sécurité existants.
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
| **Confinement, Éradication, Récupération** | Rôle variable selon l'organisation (pas une pratique universelle). Défini dans les procédures |
| **Post-incident** | Recommandations pour renforcer la posture de sécurité |

🔑 Intégrer ou séparer le hunting et l'incident handling est une **décision stratégique** propre à chaque organisation. Ils **ne fonctionnent pas toujours séparément**.

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
- **Mettre en évidence** les vulnérabilités (ex. : vulnérabilité d'escalade de privilèges, donc on cherche des anomalies de niveaux de privilège)
- **Orienter** l'usage de la threat intelligence
- **Affiner** les plans d'IR
- **Améliorer** les contrôles de sécurité

Outils cités : scanners de vulnérabilités, outils de pentest, plateformes de threat intelligence, **SIEM** (agrège et corrèle les événements).

## ✅ Questions de la page 1

| # | Question (FR) | Réponse (FR) | **Réponse à mettre sur HTB** |
|---|---|---|---|
| 1 | Le threat hunting s'utilise ... (« proactively » / « reactively » / « proactively and reactively ») | De façon proactive **et** réactive | **`proactively and reactively`** |
| 2 | Le threat hunting et l'incident handling fonctionnent toujours de manière indépendante. (True/False) | Faux | **`false`** |
| 3 | Le threat hunting et l'incident response peuvent être menés simultanément. (True/False) | Vrai | **`true`** |

### 🧭 Comment arriver aux réponses
- **Q1** : la section « Key facets » liste deux facettes : une stratégie **proactive** (hypothèses, TTPs) et une réponse **réactive** (artefacts d'un incident vérifié). Les deux existent, donc la bonne option est la troisième.
- **Q2** : le mot clé est « **always** ». La fin de la section « Relationship Between Incident Handling & Threat Hunting » dit que l'intégration ou la séparation dépend de l'organisation. Une affirmation avec « toujours » est donc fausse.
- **Q3** : la section « When Should We Hunt? » cite « During an Incident Response Activity » : on chasse en parallèle de l'IR pour trouver d'autres systèmes compromis.

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

| Question (FR) | Réponse (FR) | **Réponse à mettre sur HTB** |
|---|---|---|
| On peut formuler des hypothèses qui ne sont pas testables. (True/False) | Faux | **`false`** |

### 🧭 Comment arriver à la réponse
Dans l'étape « Formulating Hypotheses », le cours dit : *« We strive to make these hypotheses testable »* et l'exemple précise que l'hypothèse doit être **spécifique et testable**. Une hypothèse non testable ne dit pas où chercher ni quoi chercher, donc elle est **fausse** comme option.

---

# Page 3 — Threat Hunting Glossary

> Cette page ne contient **aucune question**. C'est une page de vocabulaire à connaître pour les pages suivantes et pour l'examen.

## 📘 Résumé : les définitions essentielles

| Terme | Définition en français |
|---|---|
| **Adversary** (adversaire) | Entité qui cherche à s'infiltrer dans l'organisation pour atteindre ses objectifs (gain financier, informations internes, propriété intellectuelle). Catégories : cybercriminels, menaces internes (insiders), hacktivistes, acteurs étatiques |
| **APT** (Advanced Persistent Threat) | Groupe très organisé ou étatique, avec beaucoup de ressources, actif sur de longues périodes. « Advanced » renvoie à la planification stratégique sophistiquée (pas forcément à une technique avancée). « Persistent » renvoie à leur obstination |
| **TTPs** | Signature opérationnelle d'un adversaire |
| ↳ **Tactics** | Objectifs stratégiques : le **pourquoi** |
| ↳ **Techniques** | Méthodes générales : le **comment** |
| ↳ **Procedures** | Étapes détaillées, la « recette » |
| **Indicator** | **Données + contexte = indicateur**. Une donnée technique sans contexte a peu de valeur |
| **Threat** (menace) | **Intention + Capacité + Opportunité** |
| **Campaign** | Ensemble d'incidents partageant des TTPs similaires et des objectifs comparables |
| **IOCs** (Indicators of Compromise) | Traces numériques d'une intrusion : hashes, IPs, URLs, domaines, noms d'exécutables/scripts |

### La Pyramid of Pain (David Bianco)
Plus on monte, plus l'indicateur est **difficile à obtenir pour le défenseur**, mais plus il est **coûteux à changer pour l'attaquant**.

```
            /\
           /TTPs\            ← Tough (très dur pour l'attaquant)
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

| Niveau | Pourquoi c'est fiable ou non |
|---|---|
| **Hash** | Un seul octet modifié change le hash. Trivial à changer, peu fiable |
| **IP** | VPN, proxy, TOR, spoofing. Facile à changer |
| **Domaine** | DGA (algorithmes de génération de domaines), DNS dynamique. Simple à changer |
| **Artefacts réseau/hôte** | Motifs de trafic, clés de registre, chemins de fichiers, processus. Gênant à changer sans casser l'opération |
| **Outils** | Malware, exploits, frameworks C2. Difficile à remplacer (mais les adversaires avancés les personnalisent) |
| **TTPs** | Le sommet : l'attaquant doit changer sa façon de travailler |

### Le Diamond Model (modèle du diamant)
4 sommets reliés entre eux :

| Sommet | Rôle |
|---|---|
| **Adversary** | Qui attaque |
| **Capability** | Outils, malware, exploits, TTPs |
| **Infrastructure** | Serveurs, domaines, IPs, botnets utilisés |
| **Victim** | Cible (personne, organisation, système) |

*Exemple du cours* : une institution financière (Victim) est ciblée par un groupe cybercriminel (Adversary) via du spear-phishing (Capability) envoyé depuis un botnet (Infrastructure) pour livrer un cheval de Troie bancaire.

🔑 **Comparaison** : la Cyber Kill Chain décrit les **étapes** d'une attaque. Le Diamond Model décrit les **composants** de l'intrusion et leurs relations. Les deux se complètent.

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
| Niveau | Public | Contenu | Question à laquelle il répond |
|---|---|---|---|
| **Strategic** | Dirigeants (C-suite, VPs) | Vue d'ensemble dans le temps, TTPs/MO, alignement avec les risques de l'entreprise | **Qui ? Pourquoi ?** |
| **Operational** | Management intermédiaire | Détail des campagnes, TTPs | **Comment ? Où ?** |
| **Tactical** | Défenseurs réseau / SOC | Actions immédiates, détails techniques (IPs, domaines, hashes, clés de registre, mutex) | Quoi bloquer/détecter maintenant |

*Exemples du cours* : APT28 (stratégique), campagne ransomware REvil (opérationnel), IoCs REvil (tactique).

### Comment traiter un rapport de CTI tactique (6 étapes)
1. **Comprendre** la portée et le récit du rapport (la menace nous concerne-t-elle ?)
2. **Repérer et classer** les IoCs : réseau (IPs, domaines), hôte (hashes, clés de registre), email (adresses, objets), + mutex, certificats SSL, User-Agents, etc.
3. **Comprendre le cycle de vie** de l'attaque (TTPs mappés sur **MITRE ATT&CK**)
4. **Analyser et valider** les IoCs : VirusTotal, AlienVault OTX, âge de l'IoC, contexte (une IP C2 peut aussi héberger des sites légitimes), taux de faux positifs
5. **Intégrer** dans l'infrastructure : règles pare-feu, EDR, IDS/IPS, passerelle mail. Parfois **alerter plutôt que bloquer** pour ne pas casser un service critique. Documenter via le change management
6. **Chasser** de façon proactive (IoCs + TTPs, ex. PowerShell suspect) puis **surveiller en continu** et **partager** (ISACs/ISAOs)

## ✅ Questions de la page 4

| # | Question (FR) | Réponse (FR) | **Réponse à mettre sur HTB** |
|---|---|---|---|
| 1 | Il est utile que la CTI fournisse au SOC une seule IP sans contexte. (True/False) | Faux | **`false`** |
| 2 | Quand un incident survient et que la CTI est prévenue, que doit-elle faire ? (« Do Nothing » / « Reach out to the Incident Handler/Incident Responder » / « Provide IOCs on all research being conducted, regardless if the IOC is verified ») | Contacter l'Incident Handler / Incident Responder | **`Reach out to the Incident Handler/Incident Responder`** |
| 3 | Même question avec d'autres options (« Provide IOCs on all research... regardless if verified » / « Do Nothing » / « Provide further IOCs and TTPs associated with the incident ») | Fournir d'autres IoCs et TTPs associés à l'incident | **`Provide further IOCs and TTPs associated with the incident`** |
| 4 | Une CTI bien conçue et analysée peut ... (« be used for security awareness » / « be used for fine-tuning network segmentation » / « provide insight into adversary operations ») | Donner un aperçu des opérations de l'adversaire | **`provide insight into adversary operations`** |

### 🧭 Comment arriver aux réponses
- **Q1** : formule du glossaire « **Data + contexte = indicateur** ». Une IP seule n'a pas de contexte, donc elle est peu utile. De plus, le principe d'**actionabilité** impose de donner des directives claires.
- **Q2 et Q3** : la même question avec des listes différentes. Il faut choisir **l'option qui existe dans la liste ET qui respecte les principes de la CTI** :
  - Si « Reach out to the Incident Handler » est proposé, c'est la bonne réponse (la CTI doit être en contact avec l'IR).
  - Si cette option n'est pas proposée, la bonne est « Provide further IOCs and TTPs associated with the incident ».
  - L'option « IOCs ... regardless if verified » est toujours fausse (principe d'**exactitude**). « Do Nothing » est toujours fausse.
- **Q4** : la section « Criteria Of CTI » dit que la CTI « provides visibility into adversary operations ». Les deux autres options ne figurent pas dans le cours comme apports de la CTI.

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
- Les données CTI datent de **mars 2023**, donc il faut régler la période sur **« last 15 years »**.

### ⚙️ Préparation de Kibana
1. Lancer la cible (**Spawn Target**), attendre **3 à 5 minutes**.
2. Ouvrir `http://[IP cible]:5601` → menu latéral → **Discover**.
3. Icône calendrier → **last 15 years** → **Apply**.
4. Fuseau horaire : `http://[IP cible]:5601/app/management/kibana/settings` → **Europe/Copenhagen**.

### ⌨️ Les événements Sysmon à connaître
| Event ID | Nom | Utilité dans le cours |
|---|---|---|
| **1** | Process creation | Voir les processus lancés, leurs arguments, leur parent |
| **3** | Network connection | Connexions réseau (les navigateurs sont souvent exclus de la config) |
| **11** | File create | Création de fichiers (`Zone.Identifier` = fichier venant d'Internet) |
| **15** | FileCreateStreamHash | Téléchargement depuis un navigateur |
| **22** | DNSEvent | Requêtes DNS depuis l'hôte |
| (Windows) **4624 / 4625** | Logon réussi / échoué | Brute force, mouvement latéral |

### 🔎 Déroulé de la chasse (hypothèse : phishing réussi avec un OneNote malveillant)

| Étape | Requête KQL | Ce qu'on découvre |
|---|---|---|
| 1. Téléchargement du fichier | `event.code:15 AND file.name:*invoice.one` | 3 résultats. Téléchargé par **MSEdge** dans le dossier Downloads de **Bob**. Horodatage : **26 mars 2023 @ 22:05:47** |
| 2. Confirmation | `event.code:11 AND file.name:invoice.one*` | Machine **WS001**, avec le `Zone.Identifier` (le `*` est à la fin car le nom contient `:Zone.Identifier`). IP **192.168.28.130** |
| 3. IP de WS001 | `event.code:3 AND host.hostname:WS001` (regarder `source.ip`) | Confirme l'IP |
| 4. DNS Zeek (22:05:00 à 22:05:48) | `source.ip:192.168.28.130 AND dns.question.name:*` | Filtrer le bruit (google.com, etc.). On voit **mail.google.com**, puis **file.io**, puis SmartScreen |
| 5. IPs de file.io | Champ `dns.answers.data` | `34.197.10.85`, `3.213.216.16` |
| 6. Connexions vers ces IPs | Recherche de `destination.ip` sur ces IPs | Connexions sur le port 443 : **Bob a téléchargé `invoice.one` depuis file.io** |
| 7. Ouverture du fichier | `event.code:1 AND process.command_line:*invoice.one*` | OneNote lance le fichier ~**6 secondes** après le téléchargement |
| 8. Enfants de OneNote | `event.code:1 AND process.parent.name:"ONENOTE.EXE"` | `OneNoteM.exe` (normal) et **`cmd.exe` qui exécute `invoice.bat`** |
| 9. Enfants du batch | `event.code:1 AND process.parent.command_line:*invoice.bat*` | **PowerShell** qui télécharge un script **Pastebin** (PID **9944**) |
| 10. Activité PowerShell | `process.pid:"9944" and process.name:"powershell.exe"` | 17 événements : script de brute force de mots de passe, dépôt d'un EXE, DNS **ngrok**, connexions vers le C2 (IP `18.158.249.75`), requêtes vers **DC1** |
| 11. Zeek sur l'IP C2 | `destination.ip:18.158.249.75` | Activité qui continue le lendemain. L'IP de ngrok change ensuite (`3.125.102.39`) |
| 12. Le dropper | `process.name:"default.exe"` | Exécuté. Dépose **`svchost.exe`**, **`SharpHound.exe`**, `payload.exe`, un **fichier VBS**… |
| 13. SharpHound | `process.name:"SharpHound.exe"` | Exécuté **2 fois** (~2 minutes d'écart), collecte `all` : cartographie de l'Active Directory |
| 14. Hash connu | `process.hash.sha256:018d37cb…` | Hash présent sur **WS001** et **PKI** (compromission du serveur PKI) |
| 15. Mouvement latéral | Parent = **PSEXESVC** sur PKI | Utilisation de **PsExec**. Utilisateur compromis : **svc-sql1** |
| 16. Brute force | `(event.code:4624 OR event.code:4625) AND winlog.event_data.LogonType:3 AND source.ip:192.168.28.130` | 2 échecs sur **administrator**, puis des connexions réussies de **svc-sql1** (le 28 mars) |

### Champs Kibana à ajouter en colonnes (très utile)
`process.name`, `process.args`, `process.pid`, `event.code`, `file.path`, `dns.question.name`, `destination.ip`, `source.ip`, `host.hostname`, `dns.answers.data`

🔑 **Points d'analyse à retenir**
- Le rapport CTI parle de `mega.io` et `transfer.sh`, alors que l'incident réel passe par **`file.io`**. Les IoCs d'un rapport sont donc **incomplets**, et la chasse comportementale (TTPs) trouve ce que la chasse par IoCs rate.
- Les connexions réseau des navigateurs sont exclues de la config Sysmon. Les **logs Zeek** comblent ce manque.
- Il y a un délai entre certaines actions, ce qui suggère une **intervention humaine** (pas un simple script).

## ✅ Questions de la page 5

| # | Question (FR) | Réponse (FR) | **Réponse à mettre sur HTB** |
|---|---|---|---|
| 1 | Dans la partie sur `default.exe`, un fichier VBS est mentionné. Donne son nom complet avec l'extension | Le fichier VBS est `XceGuhkzaTrOy.vbs` | **`XceGuhkzaTrOy.vbs`** |
| 2 | Stuxbot a téléversé et exécuté mimikatz. Donne les arguments du processus (ce qui suit `.\mimikatz.exe`) | Commande DCSync sur le domaine eagle.local | **`lsadump::dcsync /domain:eagle.local /all /csv, exit`** |
| 3 | Du code PowerShell chargé en mémoire scanne les partages réseau. Avec les logs PowerShell, trouve de quel outil de hacking connu il provient (format : `P____V___`) | PowerView | **`PowerView`** |

### 🧭 Comment arriver aux réponses

> ⚠️ **Honnêteté** : le cours ne détaille pas les requêtes exactes pour ces 3 questions. Les méthodes ci-dessous sont reconstituées à partir de la logique du cours (confiance : bonne pour Q1 et Q2, moyenne pour Q3). À vérifier sur la cible, car les noms de champs peuvent légèrement varier.

**Q1 — le fichier VBS**
1. Lancer la requête du cours : `process.name:"default.exe"`
2. Ajouter les colonnes `file.path`, `event.code`, `process.name`
3. Faire défiler vers le haut (le cours précise : *« If we scroll up there's further activity… including the uploading of "payload.exe", a VBS file… »*)
4. Repérer la ligne `file.path` qui se termine par `.vbs` (événements `event.code:11`, création de fichier) et recopier le nom du fichier.
- Variante plus directe : `event.code:11 AND file.name:*.vbs`

**Q2 — arguments de Mimikatz**
1. Requête : `process.name:"mimikatz.exe"`
2. Ajouter la colonne `process.args`
3. Les arguments apparaissent après `mimikatz.exe`. Recopier **exactement** la valeur (attention aux espaces et à la virgule avant `exit`).
- Ce que ça fait : `lsadump::dcsync` simule un contrôleur de domaine pour demander la réplication des hashes de mots de passe de **tous** les comptes (`/all`), au format CSV. C'est une technique de vol d'identifiants (MITRE T1003.006).

**Q3 — l'outil PowerShell (PowerView)**
1. Les logs PowerShell sont dans l'index `windows*` et correspondent au **Script Block Logging**, **event ID 4104** (le contenu des scripts exécutés, y compris ceux chargés en mémoire).
2. Requête : `event.code:4104` puis chercher des mots typiques du scan de partages, par exemple `Invoke-ShareFinder` ou `Find-DomainShare`.
3. Ouvrir le document : le code du script (champ du type `powershell.file.script_block_text` ou `message`) contient les fonctions d'origine, et l'en-tête/les noms de fonctions renvoient à **PowerView** (outil de PowerSploit pour la reconnaissance Active Directory).
- Format demandé `P____V___` : P + 4 lettres + V + 3 lettres = **PowerView**.

---

# Page 6 — Skills Assessment : Hunting For Stuxbot (Round 2)

## 📘 Résumé

### Nouvelles informations sur la dernière version de Stuxbot
1. Utilise le dossier **`C:\Users\Public`** pour déposer des outils supplémentaires (**Lateral Tool Transfer**, MITRE T1570).
2. Utilise les **clés de registre Run** pour la persistance (MITRE **T1547.001**).
3. Utilise **PowerShell Remoting** (WinRM) pour le mouvement latéral et pour atteindre les contrôleurs de domaine.

### Même environnement qu'à la page 5
Index `windows*` (audit Windows, Sysmon, PowerShell) et `zeek*`. Même préparation : **Spawn Target**, `http://[IP]:5601` → Discover → **last 15 years**.

## ✅ Les 3 hunts

> ⚠️ **Important** : le cours ne fournit **pas** les réponses de cette évaluation, et je ne les connais pas (elles dépendent de la cible). Je ne les invente pas. Voici la méthode et les requêtes de départ à adapter. Les noms de champs sont ceux demandés par l'énoncé.

### 🔎 Hunt 1 — Lateral Tool Transfer vers `C:\Users\Public`

**Question** : donner le contenu du champ `user.name` du document lié à un outil transféré dont le nom commence par **« r »**.

**Idée** : quand un outil est copié vers une autre machine via un partage SMB, la création de fichier apparaît dans Sysmon (event **11**) sur la machine cible, dans `C:\Users\Public`.

⌨️ **Requête de départ**
```
event.code:11 AND file.path:C\:\\Users\\Public\\*
```
**Étapes**
1. Lancer la requête (période : 15 ans).
2. Ajouter les colonnes : `file.name`, `file.path`, `user.name`, `host.hostname`, `process.name`.
3. Trier / chercher dans `file.name` ceux qui **commencent par « r »**. Option : ajouter `AND file.name:r*`.
4. Pour distinguer un transfert distant d'une création locale, regarder `process.name` / `process.pid` (une écriture via SMB s'affiche souvent avec le processus **System**, PID 4).
5. Ouvrir le document concerné et lire le champ **`user.name`**.

**Réponse à mettre sur HTB** : `…` *(à compléter avec ce que tu trouves sur la cible)*

### 🔎 Hunt 2 — Persistance par clés Run du registre

**Question** : donner le contenu du champ `registry.value` du document lié à la **première** action de persistance par registre.

**Idée** : Sysmon enregistre les modifications de registre (event **13**, *RegistryEvent Value Set*). On cible les clés `Run` / `RunOnce`.

⌨️ **Requête de départ**
```
event.code:13 AND registry.path:*\\CurrentVersion\\Run*
```
**Étapes**
1. Lancer la requête.
2. Ajouter les colonnes : `registry.path`, `registry.value`, `process.name`, `host.hostname`, `user.name`.
3. **Trier par date croissante** (la plus ancienne en premier), puisqu'on cherche la **première** action.
4. Ouvrir le premier document pertinent et lire `registry.value`.
5. Si rien ne sort, essayer `registry.path:*Run*` ou retirer le filtre sur `event.code`.

**Réponse à mettre sur HTB** : `…` *(à compléter)*

### 🔎 Hunt 3 — PowerShell Remoting vers DC1

**Question** : donner le contenu du champ `winlog.user.name` du document lié au mouvement latéral par PowerShell Remoting **vers DC1**.

**Idée** : un PowerShell Remoting entrant fait apparaître le processus **`wsmprovhost.exe`** sur la machine cible (ici DC1), et des connexions WinRM (ports **5985** HTTP / **5986** HTTPS).

⌨️ **Requêtes de départ** (essayer dans cet ordre)
```
event.code:1 AND process.name:"wsmprovhost.exe"
```
```
destination.port:(5985 OR 5986)
```
**Étapes**
1. Lancer la première requête et ajouter les colonnes : `host.hostname`, `process.name`, `process.parent.name`, `winlog.user.name`, `user.name`.
2. Garder les lignes où **`host.hostname`** correspond à **DC1**.
3. Lire `winlog.user.name` dans le document concerné.
4. Si la requête ne renvoie rien, utiliser la seconde (trafic WinRM dans `zeek*` ou événements Sysmon 3) pour identifier la source, puis recouper avec `event.code:4624` sur DC1.

**Réponse à mettre sur HTB** : `…` *(à compléter)*

> 💡 Si tu me donnes ce que tu trouves (ou les résultats de tes requêtes), je complète ces trois réponses et la méthode exacte.

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
| 6 | Hunt 1 (`user.name`) | *(à compléter)* |
| 6 | Hunt 2 (`registry.value`) | *(à compléter)* |
| 6 | Hunt 3 (`winlog.user.name`) | *(à compléter)* |

---

# 🧠 Cheat-sheet KQL du module

| Objectif | Requête |
|---|---|
| Téléchargement navigateur d'un fichier | `event.code:15 AND file.name:*invoice.one` |
| Création de fichier (avec Zone.Identifier) | `event.code:11 AND file.name:invoice.one*` |
| Connexions réseau d'un hôte | `event.code:3 AND host.hostname:WS001` |
| DNS Zeek d'une machine | `source.ip:192.168.28.130 AND dns.question.name:*` |
| Process lancé avec un fichier en argument | `event.code:1 AND process.command_line:*invoice.one*` |
| Enfants d'un processus parent (par nom) | `event.code:1 AND process.parent.name:"ONENOTE.EXE"` |
| Enfants d'un processus parent (par ligne de commande) | `event.code:1 AND process.parent.command_line:*invoice.bat*` |
| Activité d'un processus précis | `process.pid:"9944" and process.name:"powershell.exe"` |
| Exécution d'un binaire | `process.name:"default.exe"` |
| Recherche par hash | `process.hash.sha256:<hash en minuscules>` |
| Logons réseau réussis/échoués depuis une IP | `(event.code:4624 OR event.code:4625) AND winlog.event_data.LogonType:3 AND source.ip:192.168.28.130` |

**Syntaxe KQL à retenir** : `AND` / `OR` / `NOT`, `*` pour joker, guillemets pour une valeur exacte, `\\` pour échapper les antislashs des chemins Windows.