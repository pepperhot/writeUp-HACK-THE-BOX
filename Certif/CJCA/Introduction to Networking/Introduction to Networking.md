# 🎯 Fiche de révision HTB CJCA — Introduction to Networking

> **Légende**
> - 📘 Résumé du cours (traduit en français)
> - 🔑 À retenir absolument
> - ⌨️ Commandes / requêtes importantes
> - ✅ Question, réponse en français, et **Réponse à mettre sur HTB**
> - 🧭 Procédure détaillée, étape par étape, pour retrouver la réponse
> - ⚠️ Passage reconstitué ou erreur repérée dans le cours

> ℹ️ Ce module n'a **aucune cible** (pas de Pwnbox, pas de VPN, pas de lab). Les seules questions sont des calculs de **subnetting** (page 10). Il n'y a donc pas de bloc de connexion.

---

## Sommaire

1. [Page 1 — Networking Overview](#page-1--networking-overview)
2. [Page 2 — Network Types](#page-2--network-types)
3. [Page 3 — Networking Topologies](#page-3--networking-topologies)
4. [Page 4 — Proxies](#page-4--proxies)
5. [Page 5 — Networking Models](#page-5--networking-models)
6. [Page 6 — The OSI Model](#page-6--the-osi-model)
7. [Page 7 — The TCP/IP Model](#page-7--the-tcpip-model)
8. [Page 8 — Network Layer](#page-8--network-layer)
9. [Page 9 — IPv4 Addresses](#page-9--ipv4-addresses)
10. [Page 10 — Subnetting](#page-10--subnetting)
11. [Page 11 — MAC Addresses](#page-11--mac-addresses)
12. [Page 12 — IPv6 Addresses](#page-12--ipv6-addresses)
13. [Page 13 — Networking Key Terminology](#page-13--networking-key-terminology)
14. [Page 14 — Common Protocols](#page-14--common-protocols)
15. [Page 15 — Wireless Networks](#page-15--wireless-networks)
16. [Page 16 — Virtual Private Networks](#page-16--virtual-private-networks)
17. [Page 17 — Vendor Specific Information](#page-17--vendor-specific-information)
18. [Page 18 — Key Exchange Mechanisms](#page-18--key-exchange-mechanisms)
19. [Page 19 — Authentication Protocols](#page-19--authentication-protocols)
20. [Page 20 — TCP/UDP Connections](#page-20--tcpudp-connections)
21. [Page 21 — Cryptography](#page-21--cryptography)
22. [Mémo final : toutes les réponses HTB](#mémo-final--toutes-les-réponses-htb)
23. [Cheat-sheet du module](#cheat-sheet-du-module)

---

# Page 1 — Networking Overview

## 📘 Résumé

- Un **réseau** permet à deux ordinateurs de communiquer. Il varie par sa **topologie** (mesh, tree, star), son **support** (ethernet, fibre, coax, wireless) et ses **protocoles** (TCP, UDP, IPX).
- Un réseau **plat** (flat) est facile à monter mais peu sûr. Découper en **petits réseaux** ajoute des couches de défense : le pivot devient plus lent, plus bruyant, plus facile à détecter.
- Analogie : ACL autour des réseaux = clôture avec points d'entrée contrôlés. Cartographier et documenter = éclairage. IDS (Suricata, Snort) = buissons sous les fenêtres.

### Pourquoi un pentester doit comprendre les masques
- Beaucoup de pentesters mettent `/24` par réflexe. Un `/25` coupe le réseau en deux moitiés.
- Cas du cours : un rapport affirmait qu'un Domain Controller était hors ligne alors qu'il était simplement sur **l'autre moitié** du réseau.

| Équipement | Adresse |
|---|---|
| Server Gateway | `10.20.0.1/25` |
| Domain Controller | `10.20.0.10/25` |
| Client Gateway | `10.20.0.129/25` |
| Client Workstation | `10.20.0.200/25` |
| Pentester | `10.20.0.252/24` (gateway 10.20.0.1) |

### FQDN vs URL
| Terme | Définition | Exemple |
|---|---|---|
| **FQDN** | Adresse du « bâtiment » | `www.hackthebox.eu` |
| **URL** | Adresse + « étage, bureau, boîte aux lettres » | `https://www.hackthebox.eu/example?floor=2&office=dev&employee=17` |

Chaîne d'une requête web : client → routeur (« bureau de poste ») → **ISP** → **DNS** (annuaire : nom → IP) → serveur de destination → réponse vers l'IP source.

### Bonnes pratiques de segmentation (réseau d'entreprise idéal : 5 réseaux)
| Élément | Où le placer | Pourquoi |
|---|---|---|
| **Serveur web** | **DMZ** | Joignable depuis Internet, donc plus exposé |
| **Postes de travail** | Réseau dédié (+ firewall hôte) | Limite spoofing / MITM entre postes et serveurs |
| **Switch + routeur** | Réseau d'**administration** | Empêche d'écouter leurs échanges (ex. annonces **OSPF** forgées → MITM) |
| **Téléphones IP** | Réseau dédié | Anti-écoute + priorisation (la latence compte) |
| **Imprimantes** | Réseau dédié | Presque impossible à sécuriser, **NTLMv2** envoyé si l'imprimante demande l'authentification, persistance possible |

## 🔑 À retenir
- Segmenter = ralentir et rendre visible l'attaquant.
- Une imprimante ne doit ni parler à Internet, ni recevoir du SMB (445) des postes, ni initier des connexions vers les postes.
- Un « host down » peut juste signifier « autre sous-réseau ».

## ✅ Questions
Aucune question sur cette page.

---

# Page 2 — Network Types

## 📘 Résumé

### Termes courants
| Type | Définition |
|---|---|
| **WAN** (Wide Area Network) | Internet, ou ensemble de LAN reliés |
| **LAN** (Local Area Network) | Réseau interne (maison, bureau) |
| **WLAN** | LAN accessible en Wi-Fi |
| **VPN** | Relie plusieurs sites / un poste à un LAN |

- **WAN** : on le reconnaît à un protocole de routage WAN (**BGP**) et à des IP **hors RFC 1918**.
- **LAN/WLAN** : IP privées **RFC 1918** : `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`. WLAN = LAN sans câbles (désignation surtout sécuritaire).

### Les 3 types de VPN
| Type | Principe |
|---|---|
| **Site-to-Site** | Deux équipements réseau (routeurs/firewalls) relient des réseaux entiers |
| **Remote Access** | Le poste crée une interface virtuelle (TUN). HTB utilise **OpenVPN** |
| **SSL VPN** | Dans le navigateur (ex. **Pwnbox**), streaming d'application ou de bureau |

🔑 **Split-tunnel** : seules certaines routes passent par le VPN (ex. `10.10.10.0/24`). Pratique pour HTB (pas de surveillance d'Internet), mais mauvais pour une entreprise : un poste infecté sort sur Internet sans passer par la détection réseau.

### Termes « livre »
| Type | Définition |
|---|---|
| **GAN** (Global Area Network) | Réseau mondial (Internet, réseaux internes de multinationales) |
| **MAN** (Metropolitan Area Network) | Réseau régional reliant plusieurs LAN (fibre, routeurs haute performance) |
| **WPAN** (Wireless Personal Area Network) | Réseau personnel (Bluetooth, **Piconet**, Z-Wave, ZigBee, Insteon pour l'IoT) |

## 🔑 À retenir
- RFC 1918 : `10/8`, `172.16/12`, `192.168/16`.
- Split-tunnel vs full-tunnel : impact sur la détection.

## ✅ Questions
Aucune question sur cette page.

---

# Page 3 — Networking Topologies

## 📘 Résumé

Une topologie est **physique** (câblage) ou **logique** (circulation des données). Une topologie logique peut différer de la physique.

Trois domaines : **connexions** (câble coaxial, fibre, paire torsadée / Wi-Fi, cellulaire, satellite), **nœuds** (répéteurs, hubs, bridges, switches, routeurs/modems, gateways, firewalls), **classifications**.

| Topologie | Principe | Point clé |
|---|---|---|
| **Point-to-Point** | Lien dédié entre 2 hôtes | Ne pas confondre avec P2P |
| **Bus** | Support partagé, un seul émetteur à la fois | Pas de composant central |
| **Star** | Chaque hôte relié à un équipement central (switch, routeur, hub) | Trafic très concentré au centre |
| **Ring** | Chaque hôte relié par 2 câbles (entrée, sortie), sens unique, **token** | Une logique en anneau peut reposer sur une étoile physique |
| **Mesh** | Chaque nœud choisit ses connexions. **Fully meshed** : tout relié à tout. **Partially meshed** : liens partiels | Haute fiabilité (WAN/MAN) |
| **Tree** | Étoile étendue, hiérarchique | Grands bâtiments, MAN |
| **Hybrid** | Mélange de 2 topologies ou plus | Tree relié à un tree reste un tree |
| **Daisy Chain** | Hôtes reliés en série | Automatisation (CAN) |

## 🔑 À retenir
- 8 types : Point-to-Point, Bus, Star, Ring, Mesh, Tree, Hybrid, Daisy Chain.
- Physique ≠ logique.

## ✅ Questions
Aucune question sur cette page.

---

# Page 4 — Proxies

## 📘 Résumé

**Proxy** = équipement/service placé **au milieu** d'une connexion, qui sert de **médiateur** et doit pouvoir **inspecter** le contenu. Sans cette capacité, c'est une **gateway**. Un proxy opère presque toujours en **couche 7 (OSI)**. Un VPN qui change l'IP n'est techniquement **pas** un proxy.

| Type | Rôle | Exemples |
|---|---|---|
| **Forward / Dedicated Proxy** | Le client demande, le proxy exécute la requête (filtre le **sortant**) | Web filter d'entreprise, **Burp Suite** |
| **Reverse Proxy** | Filtre l'**entrant**, écoute et relaie vers un réseau fermé | **Cloudflare**, **ModSecurity** (WAF) |
| **Transparent Proxy** | Le client ignore son existence | Interception du trafic Internet |
| **Non-transparent Proxy** | Le client doit être configuré pour l'utiliser | Configuration proxy du navigateur |

### Points sécurité
- Les navigateurs **IE / Edge / Chrome** respectent le « System Proxy » (WinSock). **Firefox** utilise libcurl et ses propres réglages : un malware qui reprend le proxy système ne reprendra pas ceux de Firefox.
- Un malware peut utiliser le **DNS** comme C2 : détectable avec **Sysmon**.
- Pentest : un **reverse proxy** sur un poste infecté renvoie les clients vers l'attaquant (contourne firewall / IDS, trafic web dans un tunnel SSH).
- ModSecurity : lire le **Core Rule Set**. Cloudflare en WAF impose de lui laisser déchiffrer le HTTPS.

## 🔑 À retenir
- Proxy = médiateur qui inspecte, couche 7.
- Forward = sortant, Reverse = entrant.
- Transparent = invisible pour le client.

## ✅ Questions
Aucune question sur cette page.

---

# Page 5 — Networking Models

## 📘 Résumé

Deux modèles en couches décrivent le transfert des données : **OSI** (7 couches, de référence) et **TCP/IP** (4 couches, pratique).

| | **OSI (ISO/OSI)** | **TCP/IP** |
|---|---|---|
| Nature | Modèle de référence, strict (ITU + ISO) | Famille de protocoles de l'Internet |
| Souplesse | Règles strictes | Règles allégées |
| Couches | 7 | 4 |

### Encapsulation et PDU
- Chaque couche ajoute un **en-tête** au **PDU** (Protocol Data Unit) de la couche supérieure : c'est l'**encapsulation**. Le récepteur fait l'inverse (**décapsulation**).
- PDU par couche : **Data** (application) → **Segment / Datagram** (transport) → **Packet** (réseau) → **Frame** (liaison) → **Bits** (physique).

Pour un pentester : TCP/IP pour comprendre la connexion globale, OSI pour disséquer le trafic intercepté.

## 🔑 À retenir
- Encapsulation à l'envoi, décapsulation à la réception.
- Segment (L4), Packet (L3), Frame (L2), Bit (L1).

## ✅ Questions
Aucune question sur cette page.

---

# Page 6 — The OSI Model

## 📘 Résumé

| # | Couche | Fonction |
|---|---|---|
| 7 | **Application** | Entrée/sortie des données, fonctions applicatives |
| 6 | **Presentation** | Rend la représentation des données indépendante de l'application |
| 5 | **Session** | Gère la connexion logique entre 2 systèmes |
| 4 | **Transport** | Contrôle de bout en bout, détection de congestion, segmentation |
| 3 | **Network** | Routage, transmission de bout en bout |
| 2 | **Data Link** | Transmission fiable, découpage en **frames** |
| 1 | **Physical** | Signaux électriques, optiques, ondes |

- Couches **2 à 4** : orientées transport. Couches **5 à 7** : orientées application.
- Chaque couche fournit un service à celle du dessus et utilise celui du dessous.
- L'émetteur descend de 7 à 1, le récepteur remonte de 1 à 7.

🔑 Mnémotechnique (de 7 à 1) : **A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing.

## ✅ Questions
Aucune question sur cette page.

---

# Page 7 — The TCP/IP Model

## 📘 Résumé

| # | Couche | Fonction |
|---|---|---|
| 4 | **Application** | Accès aux services des autres couches, protocoles applicatifs |
| 3 | **Transport** | Sessions (**TCP**) et datagrammes (**UDP**) |
| 2 | **Internet** | Adressage, empaquetage, routage |
| 1 | **Link** | Placement des paquets sur le support, indépendant du média |

- **IP** est en couche réseau (L3 OSI), **TCP** en couche transport (L4 OSI).
- Différence avec OSI : le **nombre de couches** (certaines sont fusionnées).

| Tâche | Protocole | Rôle |
|---|---|---|
| Adressage logique | IP | Classes, subnetting, CIDR |
| Routage | IP | Prochain nœud choisi à chaque saut |
| Erreur et contrôle de flux | TCP | Connexion virtuelle, messages de contrôle |
| Support applicatif | TCP/UDP | **Ports** |
| Résolution de noms | DNS | FQDN → IP |

## ✅ Questions
Aucune question sur cette page.

---

# Page 8 — Network Layer

## 📘 Résumé

La **couche réseau (L3)** gère l'échange de **paquets**. Les paquets ne vont pas directement au destinataire : ils passent de **nœud en nœud** (routeurs) jusqu'à la cible.

Fonctions : **Logical Addressing** et **Routing**.

Protocoles courants : `IPv4/IPv6`, `IPsec`, `ICMP`, `IGMP`, `RIP`, `OSPF`.

Les paquets transférés par un routeur n'atteignent pas les couches supérieures : le routeur les réachemine vers un nouveau nœud intermédiaire.

## ✅ Questions
Aucune question sur cette page.

---

# Page 9 — IPv4 Addresses

## 📘 Résumé

- Le **MAC** identifie l'hôte dans son réseau. Pour joindre un autre réseau il faut une **IP** (partie réseau + partie hôte). Analogie : IP = adresse postale du bâtiment, MAC = étage et appartement.
- Une IPv4 = **32 bits** = 4 **octets** (0-255), en notation décimale pointée. ~4,29 milliards d'adresses. **IANA** attribue les blocs.

### Classes (historique)
| Classe | Réseau | Première | Dernière | Masque | CIDR | IP |
|---|---|---|---|---|---|---|
| A | 1.0.0.0 | 1.0.0.1 | 127.255.255.255 | 255.0.0.0 | /8 | 16 777 214 + 2 |
| B | 128.0.0.0 | 128.0.0.1 | 191.255.255.255 | 255.255.0.0 | /16 | 65 534 + 2 |
| C | 192.0.0.0 | 192.0.0.1 | 223.255.255.255 | 255.255.255.0 | /24 | 254 + 2 |
| D | 224.0.0.0 | 224.0.0.1 | 239.255.255.255 | Multicast | | |
| E | 240.0.0.0 | 240.0.0.1 | 255.255.255.255 | Réservé | | |

- Les **2 IP en plus** (`+ 2`) sont l'adresse **réseau** et l'adresse de **broadcast**.
- **Default gateway** = IP du routeur. Souvent la première ou dernière IP du sous-réseau (usage, pas obligation technique).
- **Broadcast** = dernière adresse du sous-réseau, message à tous.

### Binaire → décimal
Valeurs des bits d'un octet : `128 64 32 16 8 4 2 1`

| Octet | Binaire | Décimal |
|---|---|---|
| 1er | `1100 0000` | 192 |
| 2e | `1010 1000` | 168 |
| 3e | `0000 1010` | 10 |
| 4e | `0010 0111` | 39 |

→ `192.168.10.39`. Masque `255.255.255.0` = `1111 1111.1111 1111.1111 1111.0000 0000`.

### CIDR
Le suffixe CIDR = **nombre de bits à 1** du masque. `192.168.10.39/24` ⇔ masque `255.255.255.0`.

## 🔑 À retenir
- IPv4 = 32 bits, 4 octets.
- CIDR `/n` = n bits réseau.
- 1ère adresse = réseau, dernière = broadcast.

> ⚠️ **Incohérence du cours** : le tableau indique « Last Address 127.255.255.255 » pour la classe A, ce qui est la dernière adresse de la plage entière (broadcast), conforme au tableau mais ne correspond pas à « 1.0.0.1 → dernier hôte ».

## ✅ Questions
Aucune question sur cette page.

---

# Page 10 — Subnetting

## 📘 Résumé

**Subnetting** = découper une plage IPv4 en plages plus petites. Pour un sous-réseau on cherche : adresse réseau, broadcast, premier hôte, dernier hôte, nombre d'hôtes.

### Méthode de base (exemple `192.168.12.160/26`)
| Détail | Valeur |
|---|---|
| Masque | `255.255.255.192` (`1111 1111.1111 1111.1111 1111.11\|00 0000`) |
| Bits hôte | 6 (les 6 derniers bits) |
| Adresse réseau | tous les bits hôte à **0** → `192.168.12.128` |
| Broadcast | tous les bits hôte à **1** → `192.168.12.191` |
| Hôtes utilisables | 64 − 2 = **62** (`.129` à `.190`) |

### Découper un sous-réseau en N sous-réseaux
1. N doit être une **puissance de 2** (2^x). x = nombre de bits à **ajouter** au masque.
2. Nouveau CIDR = ancien CIDR + x.
3. Taille de chaque sous-réseau = taille totale / N.
4. On part de l'adresse réseau et on ajoute la taille à chaque fois.

Exemple : `192.168.12.128/26` en 4 sous-réseaux → `/28`, 16 adresses chacun :

| N° | Réseau | 1er hôte | Dernier hôte | Broadcast | CIDR |
|---|---|---|---|---|---|
| 1 | 192.168.12.128 | .129 | .142 | .143 | /28 |
| 2 | 192.168.12.144 | .145 | .158 | .159 | /28 |
| 3 | 192.168.12.160 | .161 | .174 | .175 | /28 |
| 4 | 192.168.12.176 | .177 | .190 | .191 | /28 |

### Subnetting mental
1. Trouve l'**octet qui change** : `/8` → 1er, `/16` → 2e, `/24` → 3e, `/32` → 4e.
2. Calcule `CIDR % 8` (reste). Ce reste donne la taille du bloc dans l'octet :

| Reste | Taille du bloc | Forme |
|---|---|---|
| 0 | 256 | 2^8 |
| 1 | 128 | 2^7 |
| 2 | 64 | 2^6 |
| 3 | 32 | 2^5 |
| 4 | 16 | 2^4 |
| 5 | 8 | 2^3 |
| 6 | 4 | 2^2 |
| 7 | 2 | 2^1 |

3. Le zéro compte comme un nombre : `/25` → blocs de 128 → `192.168.1.0-127` (réseau `.0`, broadcast `.127`, hôtes `.1-.126`), puis `.128-255` (hôtes `.129-.254`).

### ⌨️ Aide-mémoire rapide
| CIDR | Masque (4e octet) | Taille du bloc | Hôtes utilisables |
|---|---|---|---|
| /24 | 255.255.255.0 | 256 | 254 |
| /25 | .128 | 128 | 126 |
| /26 | .192 | 64 | 62 |
| /27 | .224 | 32 | 30 |
| /28 | .240 | 16 | 14 |
| /29 | .248 | 8 | 6 |
| /30 | .252 | 4 | 2 |

## 🔑 À retenir
- Hôtes utilisables = 2^(bits hôte) − 2.
- Découper en 2^x sous-réseaux = ajouter x bits au masque.
- Bloc d'un octet = 256 / 2^(CIDR % 8).

## ✅ Questions de la page 10

### ❓ Question 1 — Masque décimal d'un /27

| | |
|---|---|
| **Question (EN)** | Submit the decimal representation of the subnet mask from the following CIDR: 10.200.20.0/27 |
| **Question (FR)** | Donne la représentation décimale du masque de sous-réseau du CIDR `10.200.20.0/27`. |
| **Réponse (FR)** | `/27` = 27 bits à 1 → 255.255.255.224 |
| **Réponse à mettre sur HTB** | `255.255.255.224` |

**🧭 Procédure complète** (calcul, aucune cible)

1. Écris les 27 bits à 1 en 4 octets : `11111111.11111111.11111111.11100000`.
   - 3 octets pleins = 24 bits. Il reste 27 − 24 = **3 bits** dans le 4e octet.
2. 4e octet = `1110 0000` = 128 + 64 + 32 = **224**.
3. Masque : `255.255.255.224`.
4. Saisis exactement : `255.255.255.224`

✔️ **Vérification** : bloc de 2^5 = 32 adresses (`256 − 224 = 32`). Cohérent avec `/27` (5 bits hôte).

---

### ❓ Question 2 — Broadcast d'un /27

| | |
|---|---|
| **Question (EN)** | Submit the broadcast address of the following CIDR: 10.200.20.0/27 |
| **Question (FR)** | Donne l'adresse de broadcast du CIDR `10.200.20.0/27`. |
| **Réponse (FR)** | Réseau `10.200.20.0`, bloc de 32 → dernière adresse `10.200.20.31` |
| **Réponse à mettre sur HTB** | `10.200.20.31` |

**🧭 Procédure complète**

1. `/27` → 5 bits hôte → bloc de 2^5 = **32** adresses.
2. Adresse réseau : `10.200.20.0` (bits hôte à 0).
3. Broadcast = bits hôte à 1 : `0 + 32 − 1 = 31` → `10.200.20.31`.
4. Plage des hôtes : `10.200.20.1` à `10.200.20.30` (30 hôtes).
5. Saisis exactement : `10.200.20.31`

---

### ❓ Question 3 — Réseau du 3e sous-réseau

| | |
|---|---|
| **Question (EN)** | Split the network 10.200.20.0/27 into 4 subnets and submit the network address of the 3rd subnet as the answer. |
| **Question (FR)** | Découpe `10.200.20.0/27` en 4 sous-réseaux et donne l'adresse réseau du **3e**. |
| **Réponse (FR)** | 4 sous-réseaux `/29` de 8 adresses : `.0`, `.8`, `.16`, `.24` → le 3e commence à `.16` |
| **Réponse à mettre sur HTB** | `10.200.20.16` |

**🧭 Procédure complète**

1. 4 = 2^2 → on ajoute **2 bits** : `/27` + 2 = **/29**.
2. Taille de chaque sous-réseau : 32 / 4 = **8** adresses.
3. Liste (on part de `10.200.20.0` et on ajoute 8) :

| N° | Réseau | Hôtes | Broadcast |
|---|---|---|---|
| 1 | 10.200.20.0/29 | .1 à .6 | 10.200.20.7 |
| 2 | 10.200.20.8/29 | .9 à .14 | 10.200.20.15 |
| 3 | 10.200.20.16/29 | .17 à .22 | 10.200.20.23 |
| 4 | 10.200.20.24/29 | .25 à .30 | 10.200.20.31 |

4. 3e sous-réseau → `10.200.20.16`.
5. Saisis exactement : `10.200.20.16`

---

### ❓ Question 4 — Broadcast du 2e sous-réseau

| | |
|---|---|
| **Question (EN)** | Split the network 10.200.20.0/27 into 4 subnets and submit the broadcast address of the 2nd subnet as the answer. |
| **Question (FR)** | Découpe `10.200.20.0/27` en 4 sous-réseaux et donne l'adresse de broadcast du **2e**. |
| **Réponse (FR)** | 2e sous-réseau `10.200.20.8/29` → broadcast `10.200.20.15` |
| **Réponse à mettre sur HTB** | `10.200.20.15` |

**🧭 Procédure complète**

1. Reprends le tableau de la question 3 : 4 sous-réseaux `/29` de 8 adresses.
2. 2e sous-réseau : réseau `10.200.20.8`.
3. Broadcast = réseau + 8 − 1 = `8 + 7 = 15` → `10.200.20.15`.
4. Saisis exactement : `10.200.20.15`

✔️ **Contrôle** : le broadcast d'un sous-réseau est toujours l'adresse juste avant le réseau du suivant (suivant = `.16`, donc `.15`).

---

# Page 11 — MAC Addresses

## 📘 Résumé

- **MAC** = adresse physique **48 bits (6 octets)**, en hexadécimal. Standards : Ethernet (802.3), Bluetooth (802.15), WLAN (802.11).
- Fixée par le fabricant, mais **modifiable** (au moins temporairement).
- Formats : `DE:AD:BE:EF:13:37`, `DE-AD-BE-EF-13-37`, `DEAD.BEEF.1337`.

### Structure
| Partie | Taille | Rôle |
|---|---|---|
| **OUI** (Organization Unique Identifier) | 3 premiers octets | Attribué par l'**IEEE** au fabricant |
| **NIC** (Individual Address Part) | 3 derniers octets | Attribué par le fabricant |

### Bits spéciaux du 1er octet
| Bit | Valeur | Signification |
|---|---|---|
| Dernier bit (bit I/G) | 0 / 1 | **Unicast** / **Multicast** |
| Avant-dernier bit (bit U/L) | 0 / 1 | **Global OUI** (IEEE) / **Localement administrée** |

| Type | Exemple |
|---|---|
| Multicast | `01:00:5E:EF:13:37` |
| Broadcast | `FF:FF:FF:FF:FF:FF` |
| Plages locales | `02:…`, `06:…`, `0A:…`, `0E:…` |

Même sous-réseau : livraison directe vers la MAC. Autre sous-réseau : la trame est adressée à la MAC de la **default gateway**.

### ARP (Address Resolution Protocol)
- Résout une **IP (L3) → MAC (L2)** sur un LAN (IPv4).
- **ARP Request** : broadcast « Who has IP ? Tell IP ». **ARP Reply** : « IP is at MAC ».

```text
1  10.129.12.100 -> 10.129.12.255 ARP  Who has 10.129.12.101?  Tell 10.129.12.100
2  10.129.12.101 -> 10.129.12.100 ARP  10.129.12.101 is at AA:AA:AA:AA:AA:AA
```

### Attaques liées aux MAC
| Attaque | Principe |
|---|---|
| **MAC spoofing** | Prendre la MAC d'un autre équipement |
| **MAC flooding** | Saturer la table MAC du switch avec de fausses adresses |
| **Contournement du MAC filtering** | Usurper une MAC autorisée |
| **ARP spoofing / poisoning** | Envoyer de fausses réponses ARP pour associer **sa** MAC à l'IP d'un autre (MITM). Outils : **Ettercap**, **Cain & Abel** |

Défenses : segmentation, authentification forte, protocoles sécurisés (IPsec, SSL), firewalls, IDS.

> ⚠️ **Erreur repérée dans le cours** : l'exemple d'ARP spoofing présente `10.129.12.255` comme IP de la passerelle, or `.255` est l'adresse de **broadcast** d'un /24. À lire comme « IP de la passerelle ».

## 🔑 À retenir
- MAC = 48 bits, OUI (3 octets) + NIC (3 octets).
- ARP : IP → MAC. ARP spoofing = MITM sur le LAN.

## ✅ Questions
Aucune question sur cette page.

---

# Page 12 — IPv6 Addresses

## 📘 Résumé

- IPv6 = **128 bits**, notation hexadécimale. **IANA** attribue les préfixes. **Dual Stack** = IPv4 et IPv6 en parallèle.
- Pas de NAT nécessaire (principe de bout en bout). Une interface peut avoir **plusieurs** adresses IPv6.

### IPv4 vs IPv6
| | IPv4 | IPv6 |
|---|---|---|
| Taille | 32 bits | 128 bits |
| Couche OSI | Réseau | Réseau |
| Espace | ~4,3 milliards | ~340 undécillions |
| Représentation | Décimale | Hexadécimale |
| Exemple de préfixe | `10.10.10.0/24` | `fe80::dd80:b1a9:6687:2d3b/64` |
| Adressage dynamique | DHCP | **SLAAC** / DHCPv6 |
| IPsec | Optionnel | Annoncé « obligatoire » par le cours |

### Types d'adresses
| Type | Description |
|---|---|
| **Unicast** | Une seule interface |
| **Anycast** | Plusieurs interfaces, une seule reçoit (la plus proche) |
| **Multicast** | Plusieurs interfaces, toutes reçoivent |

🔑 **Plus de broadcast en IPv6** : remplacé par le multicast.

### Hexadécimal
Un chiffre hex = 4 bits (0-F). Exemple : `192.168.12.160` → `C0.A8.0C.A0`.

| Décimal | Hex | Binaire |
|---|---|---|
| 10 | A | 1010 |
| 11 | B | 1011 |
| 12 | C | 1100 |
| 13 | D | 1101 |
| 14 | E | 1110 |
| 15 | F | 1111 |

### Structure d'une adresse IPv6
- 128 bits = **8 blocs** de 16 bits (4 chiffres hex), séparés par `:`.
- **Network Prefix** (réseau) + **Interface Identifier** (hôte, dérivé de la MAC 48 bits étendue à 64 bits). Préfixe par défaut **/64** ; autres : /32, /48, /56.

| Forme | Adresse |
|---|---|
| Complète | `fe80:0000:0000:0000:dd80:b1a9:6687:2d3b/64` |
| Abrégée | `fe80::dd80:b1a9:6687:2d3b/64` |

### Règles de la RFC 5952
1. Lettres **toujours en minuscules**.
2. **Zéros de tête** d'un bloc omis.
3. Une suite de blocs de zéros remplacée par `::`.
4. `::` utilisable **une seule fois** (en partant de la gauche).

> ⚠️ **Nuance** : dans la RFC 4301 / 6434, IPsec est **recommandé** et non obligatoire en IPv6. Le cours le présente comme « Mandatory ».

## ✅ Questions
Aucune question sur cette page.

---

# Page 13 — Networking Key Terminology

## 📘 Résumé : glossaire de référence

| Acronyme | Nom | Description |
|---|---|---|
| **WEP** | Wired Equivalent Privacy | Ancienne sécurité Wi-Fi |
| **WPA** | Wi-Fi Protected Access | Sécurité Wi-Fi par mot de passe |
| **TKIP** | Temporal Key Integrity Protocol | Sécurité Wi-Fi, moins sûre |
| **SSH** | Secure Shell | Connexion et exécution à distance sécurisées |
| **FTP** | File Transfer Protocol | Transfert de fichiers |
| **SMTP** | Simple Mail Transfer Protocol | Envoi/réception d'e-mails |
| **HTTP** | Hypertext Transfer Protocol | Client-serveur web |
| **SMB** | Server Message Block | Partage de fichiers, imprimantes |
| **NFS** | Network File System | Accès aux fichiers via le réseau |
| **SNMP** | Simple Network Management Protocol | Gestion d'équipements |
| **NTP** | Network Time Protocol | Synchronisation d'horloges |
| **VLAN** | Virtual Local Area Network | Segmentation logique |
| **VTP** | VLAN Trunking Protocol | Maintien des VLAN sur plusieurs switches (L2) |
| **RIP** | Routing Information Protocol | Routage à vecteur de distance |
| **OSPF** | Open Shortest Path First | IGP, routage dans un système autonome |
| **IGRP / EIGRP** | (Enhanced) Interior Gateway Routing Protocol | Routage propriétaire Cisco |
| **PGP** | Pretty Good Privacy | Chiffrement mails/fichiers |
| **NNTP** | Network News Transfer Protocol | Newsgroups |
| **CDP** | Cisco Discovery Protocol | Découverte des équipements Cisco |
| **HSRP** | Hot Standby Router Protocol | Redondance de routeurs (Cisco) |
| **VRRP** | Virtual Router Redundancy Protocol | Attribution automatique de routeurs |
| **STP** | Spanning Tree Protocol | Topologie sans boucle (L2) |
| **TACACS** | Terminal Access Controller Access-Control System | AAA centralisé |
| **SIP** | Session Initiation Protocol | Signalisation voix/vidéo |
| **VOIP** | Voice Over IP | Téléphonie sur Internet |
| **EAP / LEAP / PEAP** | Extensible Authentication Protocol et variantes | Authentification (voir page 19) |
| **SMS** | Systems Management Server | Gestion de parc Microsoft |
| **MBSA** | Microsoft Baseline Security Analyzer | Scan de vulnérabilités Windows |
| **SCADA** | Supervisory Control and Data Acquisition | Contrôle de processus industriels |
| **VPN** | Virtual Private Network | Connexion chiffrée vers un réseau |
| **IPsec** | Internet Protocol Security | Communication chiffrée (VPN) |
| **PPTP** | Point-to-Point Tunneling Protocol | Tunnel VPN (obsolète) |
| **NAT** | Network Address Translation | Plusieurs IP privées vers une IP publique |
| **CRLF** | Carriage Return Line Feed | Fin de ligne (`\r\n`) |
| **AJAX** | Asynchronous JavaScript and XML | Pages web dynamiques |
| **ISAPI** | Internet Server Application Programming Interface | Extensions de serveur web |
| **URI / URL** | Uniform Resource Identifier / Locator | Identifiant / adresse d'une ressource |
| **IKE** | Internet Key Exchange | Connexion sécurisée entre 2 machines (VPN) |
| **GRE** | Generic Routing Encapsulation | Encapsulation dans un tunnel VPN |
| **RSH** | Remote Shell | Commandes à distance sous Unix |

## ✅ Questions
Aucune question sur cette page.

---

# Page 14 — Common Protocols

## 📘 Résumé

Les protocoles sont définis par des **RFC**. Deux modes de transport : **TCP** (orienté connexion, *Three-Way Handshake*, fiable, plus lent) et **UDP** (sans connexion, rapide, sans garantie).

### TCP — ports à connaître
| Protocole | Port | Usage |
|---|---|---|
| FTP | 20-21 | Transfert de fichiers |
| SSH / SCP | 22 | Accès distant sécurisé / copie sécurisée |
| Telnet | 23 | Accès distant en clair |
| SMTP | 25 | Envoi d'e-mails |
| DNS | 53 | Résolution de noms |
| TFTP | 69 | Transfert de fichiers (UDP principalement) |
| HTTP | 80 | Web |
| Kerberos | 88 | Authentification |
| POP3 | 110 | Récupération d'e-mails |
| RPC | 111 / 135 | Appel de procédures distantes |
| NTP | 123 | Horloges |
| NetBIOS/RPC | 137-139 | Services Windows |
| IMAP | 143 | Accès aux e-mails |
| SNMP | 161-162 | Gestion réseau |
| LDAP | 389 | Annuaire |
| HTTPS | 443 | Web chiffré |
| SMB | 445 | Partage Windows |
| REXEC / RLOGIN | 512 / 513 | Exécution / shell distant |
| ISAKMP | 500 | VPN |
| NFS | 2049 | Montage distant |
| PPTP | 1723 | VPN |
| MS-SQL | 1433 | SQL Server |
| Oracle TNS | 1521 / 1526 | Listener Oracle |
| ingreslock | 1524 | Base Ingres, souvent détourné en backdoor |
| Squid proxy | 3128 | Proxy cache HTTP |
| RDP | 3389 | Bureau à distance |
| SIP | 5060 | VoIP |
| X11 | 6000 | Interface graphique distante |
| DB2 | 50000 | SGBD d'entreprise |

### UDP — ports à connaître
| Protocole | Port | Usage |
|---|---|---|
| DNS | 53 | Résolution |
| DHCP / BOOTP | 67 / 68 | Attribution d'IP |
| TFTP | 69 | Fichiers |
| NTP | 123 | Horloges |
| NetBIOS-NS | 137 | Noms NetBIOS |
| SNMP | 161 | Supervision |
| XDMCP | 177 | Login X11 distant |
| IRC | 194 | Chat |
| IKE / IPsec | 500 | VPN |
| Syslog | 514 | Logs |
| RIP | 520 | Routage |
| UPnP | 1900 | Découverte d'équipements |
| MS-SQL Browser | 1434 | Service SQL Browser |
| MySQL | 3306 | Base de données |
| RDP (Terminal Server) | 3389 | Bureau à distance |
| PostgreSQL | 5432 | Base de données |
| VNC | 5900 | Bureau partagé |

### ICMP (couche 3)
- Sert au **diagnostic et aux erreurs**. **ICMPv4** (IPv4) / **ICMPv6** (IPv6).

| Requête | Rôle |
|---|---|
| Echo Request | Teste l'accessibilité (`ping`, `traceroute`, `tracert`) |
| Timestamp Request | Heure d'un hôte distant |
| Address Mask Request | Masque d'un hôte |

| Message | Rôle |
|---|---|
| Echo Reply | Réponse à l'echo |
| Destination Unreachable | Livraison impossible |
| Redirect | Le routeur indique un meilleur routeur |
| Time Exceeded | TTL tombé à 0 |
| Parameter Problem | En-tête invalide |
| Source Quench | Ralentir l'émetteur |

### TTL et détection d'OS
Le **TTL** (Time To Live) baisse de **1 à chaque routeur**. À 0, le paquet est jeté et un *Time Exceeded* est renvoyé.

| TTL par défaut | Système probable |
|---|---|
| **128** | Windows |
| **64** | Linux / macOS |
| **255** | Solaris |

Exemple : TTL reçu **122** → Windows (128) à **6 sauts**. ⚠️ Valeur modifiable, donc indice et non preuve.

### VoIP et SIP
- Ports SIP : **5060/5061**. **H.323** : TCP/1720.
- Méthodes SIP : `INVITE`, `ACK`, `BYE`, `CANCEL`, `REGISTER`, `OPTIONS`.
- **Information disclosure** : la méthode `OPTIONS` permet d'**énumérer des utilisateurs** et les capacités du serveur. Un fichier `SEPxxxx.cnf` (config de téléphone Cisco, modèle, firmware, réseau) peut être retrouvé.

> ⚠️ **Erreurs repérées dans le cours** : `TCP Wrappers` est donné au port 113 (c'est le port d'`Ident`), et `IKE` au port 11371 dans le tableau UDP (11371 est le port d'**OpenPGP**, IKE est en 500).

## 🔑 À retenir
- Savoir par cœur : 21, 22, 23, 25, 53, 80, 110, 139/445, 143, 161, 389, 443, 3306, 3389.
- TTL 128 = Windows, 64 = Linux.
- SIP `OPTIONS` = énumération.

## ✅ Questions
Aucune question sur cette page.

---

# Page 15 — Wireless Networks

## 📘 Résumé

- Le Wi-Fi utilise des **ondes radio** en **2,4 GHz** ou **5 GHz**. Le **WAP** (Wireless Access Point) relie le Wi-Fi au réseau filaire. WWAN = réseaux cellulaires (3G, 4G LTE, 5G).
- Norme : **IEEE 802.11**. Pour se connecter : SSID + mot de passe.

### Connexion Wi-Fi
Le client envoie une **association request** : MAC, SSID, débits, canaux, protocoles de sécurité (WPA2/WPA3). Un SSID masqué reste visible dans les paquets d'authentification.

### Handshake challenge-response WEP
| Étape | Qui | Action |
|---|---|---|
| 1 | Client | Association request |
| 2 | WAP | Association response + **challenge** |
| 3 | Client | Réponse calculée avec la clé partagée |
| 4 | WAP | Vérifie et renvoie la réponse d'authentification |

**Faille du CRC** : le checksum CRC est calculé sur le texte **en clair** et inclus dans le paquet, ce qui permet de déchiffrer un paquet sans connaître la clé.

### Protections et chiffrement
| Fonction | Rôle |
|---|---|
| Chiffrement | WEP, WPA2, WPA3 |
| Contrôle d'accès | Mot de passe, filtrage MAC |
| Firewall | Règles entrantes/sortantes |

| Protocole | IV | Clé secrète | Remarque |
|---|---|---|---|
| **WEP-40 / WEP-64** | 24 bits | 40 bits | RC4, cassable |
| **WEP-104** | 24 bits | 104 bits* | RC4, cassable |

\* Le cours indique 80 bits.

> ⚠️ **Erreur repérée dans le cours** : WEP-104 utilise une clé secrète de **104 bits** (IV 24 bits + 104 = 128 bits au total), pas 80 bits. Le point à retenir reste : l'**IV de 24 bits est trop court**, d'où les collisions et la récupération de clé.

### WPA
- **WPA-Personal** (PSK, maison/petites structures) et **WPA-Enterprise** (serveur **RADIUS** / **TACACS+**, 802.1X). Utiliser **WPA2 ou WPA3** au minimum.

### LEAP vs PEAP
| | **LEAP** | **PEAP** |
|---|---|---|
| Base | EAP (Cisco) | EAP + **TLS** |
| Sécurité | Faible, attaques par dictionnaire | Tunnel TLS, certificat serveur, MSCHAPv2 protégé |
| Chiffrement | RC4 | AES / 3DES |

### TACACS+
Protocole d'authentification/autorisation des accès aux équipements réseau. La requête du WAP est **chiffrée** (SSL/TLS ou IPsec) pour protéger identifiants et éviter la falsification.

### Disassociation attack
Envoi de **trames de désassociation** pour déconnecter les clients du WAP. Sert à perturber le service ou de **prélude à un MITM** (le client se reconnecte, on capture le handshake ou on l'attire vers un faux point d'accès).

### Hardening Wi-Fi
| Mesure | Détail |
|---|---|
| **Désactiver le broadcast du SSID** | Plus de beacon frames visibles |
| **WPA2/WPA3** | Chiffrement et authentification forts |
| **MAC filtering** | Liste de MAC autorisées (contournable par spoofing) |
| **EAP-TLS** | Certificats + PKI, authentification mutuelle |

## 🔑 À retenir
- WEP = RC4 + IV de 24 bits, obsolète.
- PEAP > LEAP. EAP-TLS = référence.
- Disassociation = déconnexion forcée.

## ✅ Questions
Aucune question sur cette page.

---

# Page 16 — Virtual Private Networks

## 📘 Résumé

Un **VPN** crée une connexion **chiffrée** entre un appareil distant et un réseau privé. Usages : accès à distance sécurisé, économique (via Internet), interconnexion de sites.

Ports : **TCP/1723** (PPTP), **UDP/500** (IKEv1 / IKEv2).

| Composant | Rôle |
|---|---|
| **VPN Client** | Installé sur l'appareil (ex. client OpenVPN) |
| **VPN Server** | Accepte les connexions et route vers le réseau privé |
| **Chiffrement** | AES, IPsec |
| **Authentification** | Secret partagé, certificat |

### IPsec
| Protocole | Rôle |
|---|---|
| **AH** (Authentication Header) | Intégrité et authenticité, **pas de chiffrement** |
| **ESP** (Encapsulating Security Payload) | **Chiffrement** + authentification optionnelle |

| Mode | Principe | Usage |
|---|---|---|
| **Transport** | Chiffre la charge utile, **pas** l'en-tête IP | Hôte à hôte |
| **Tunnel** | Chiffre **tout** le paquet IP | Réseau à réseau (VPN) |

### Flux à autoriser sur un firewall pour IPsec
| Protocole | Port | Rôle |
|---|---|---|
| IP | Protocoles **50** (ESP) et **51** (AH) | Transport des en-têtes IPsec |
| **IKE** | **UDP/500** | Négociation des clés (Diffie-Hellman) |
| **ESP** | IP 50 (ou **UDP/4500** avec NAT-T) | Chiffrement du trafic |

### PPTP
Tunnel VPN issu de PPP. **Obsolète** : l'authentification **MSCHAPv2** repose sur du **DES**, cassable. Remplacé par L2TP/IPsec, IPsec/IKEv2, OpenVPN.

## 🔑 À retenir
- AH = authentifie seul. ESP = chiffre.
- Transport = hôte à hôte. Tunnel = réseau à réseau.
- IKE = UDP/500. ESP = protocole 50. NAT-T = UDP/4500.
- PPTP = TCP/1723, à éviter.

## ✅ Questions
Aucune question sur cette page.

---

# Page 17 — Vendor Specific Information

## 📘 Résumé

### Cisco IOS
Système d'exploitation des routeurs et switches Cisco : IPv6, **QoS**, chiffrement/authentification, **VPLS**, **VRF**. Géré en **CLI** (principal) ou GUI. Gère routage (OSPF, BGP), commutation (VTP, STP), services (DHCP), sécurité (**ACL**).

Accès distant par SSH ou Telnet. Le message **`User Access Verification`** identifie un équipement Cisco IOS :

```text
$ telnet 10.129.10.2
User Access Verification
Password:
```

| Mot de passe | Usage |
|---|---|
| **User** | Connexion à l'équipement |
| **Enable Password** | Passage en mode « enable » (fonctions avancées) |
| **Secret** | Accès à certains services/outils distants |
| **Enable Secret** | Mode enable, **stocké chiffré** |

### VLAN (Virtual LAN)
Regroupement logique de ports de switch = **domaine de broadcast** distinct, avec son propre sous-réseau.

| Département | VLAN | Sous-réseau |
|---|---|---|
| Servers | 10 | 192.168.1.0/24 |
| C-Level | 20 | 192.168.2.0/24 |
| Finance | 30 | 192.168.3.0/24 |
| HR | 40 | 192.168.4.0/24 |
| Marketing | 50 | 192.168.5.0/24 |
| Support | 60 | 192.168.6.0/24 |

Avantages : organisation, **sécurité** (pas de sniffing inter-VLAN), administration simplifiée, performance.

| Plage d'ID | Détail |
|---|---|
| 0 et 4095 | Réservés |
| 1 | VLAN par défaut (ne pas modifier/supprimer) |
| 1-1005 | Plage **normale** (stockée dans `vlan.dat`) ; 1002-1005 pour Token Ring/FDDI |
| 1006-4094 | Plage **étendue** (non sauvegardée dans `vlan.dat`) |

### Appartenance et types de ports
| Mode | Principe | Sécurité |
|---|---|---|
| **Statique** | Port affecté manuellement | Plus sûr |
| **Dynamique** | Selon la MAC (service **VMPS**) | Contournable par **macchanger** (spoofing MAC) |

| Port | Rôle |
|---|---|
| **Access** | Un seul VLAN (parfois un 2e pour la voix) |
| **Trunk** | Plusieurs VLAN en même temps (entre switches / switch-routeur) |

### Identification des VLAN
| Méthode | Détail |
|---|---|
| **ISL** | Cisco, propriétaire, obsolète (en-tête 26 octets + trailer 4 octets) |
| **IEEE 802.1Q** | Standard (1998). Ajoute **TPID** (`0x8100`) + **TCI** (PCP, DEI, **VID** 12 bits) |

- VID de 12 bits → 2^12 = 4096 valeurs, **4094 utilisables**.
- **Double tagging** (802.1ad) = plusieurs tags 802.1Q dans une même trame.

### ⌨️ Configurer un VLAN
**Linux**
```bash
sudo modprobe 8021q                                   # charge le module 802.1Q
lsmod | grep 8021                                     # vérifie le chargement
ip a                                                  # repère l'interface (eth0)
sudo ip link add link eth0 name eth0.20 type vlan id 20   # crée eth0.20 (VLAN 20)
sudo ip addr add 192.168.1.1/24 dev eth0.20           # affecte une IP
sudo ip link set up eth0.20                           # active l'interface
ip a | grep eth0.20                                   # contrôle
```
(`vconfig add eth0 20` est **déprécié**.)

**Windows (PowerShell)**
```powershell
Get-NetAdapter | Format-Table -AutoSize                      # liste les cartes
Get-NetAdapterAdvancedProperty -DisplayName "vlan id"        # lit le VLAN ID
Set-NetAdapter -Name "Ethernet 2" -VlanID 10                 # définit le VLAN ID
```
GUI : Gestionnaire de périphériques → Propriétés de la carte → onglet **Avancé** → **VLAN ID**. Nécessite une carte compatible.

**Analyse Wireshark / tshark**
```text
vlan               # filtre Wireshark : trames 802.1Q
vlan.id == 10      # trames du VLAN 10
```
```bash
tshark -r "capture.pcapng" -T fields -e vlan.id | sort -n -u   # liste les VLAN ID présents
```

### Attaques VLAN
| Attaque | Principe | Condition |
|---|---|---|
| **VLAN Hopping** (DTP) | L'attaquant imite un switch (**DTP**, Dynamic Trunking Protocol) et obtient un **trunk** → voit tous les VLAN. Outil : **Yersinia** | Port avec DTP activé |
| **Double tagging** | Trame avec 2 tags 802.1Q : le switch retire le 1er (VLAN natif), le 2e route vers le VLAN cible. Outils : **Scapy**, Yersinia | Attaquant dans le **VLAN natif** du trunk ; envoi **unidirectionnel** |

Défenses : désactiver DTP, ne pas laisser de VLAN natif utilisé par des ports d'accès, affectation statique, VLAN natif dédié.

### VXLAN
**Virtual eXtensible LAN** (RFC 7348) : overlay **L2 sur L3** pour les data centers multi-tenants. **VNI de 24 bits** → ~**16 millions** de segments (contre 4094 VLAN). Évite les limites de **STP** (liens bloqués).

### CDP et STP (protocoles de couche 2)
| Protocole | Rôle | Informations visibles |
|---|---|---|
| **CDP** | Découverte des équipements Cisco voisins | Device-ID, IP, port, capacités, version IOS, plateforme |
| **STP** | Empêche les boucles dans un réseau à liens redondants | Root ID, bridge ID, coûts, timers |

CDP expose beaucoup d'informations : à désactiver si inutile.

## 🔑 À retenir
- `User Access Verification` = Cisco IOS.
- Enable Secret est chiffré, Enable Password non (en pratique).
- 802.1Q : TPID `0x8100`, VID 12 bits, 4094 VLAN.
- VLAN hopping : DTP et double tagging.
- VXLAN : VNI 24 bits.

## ✅ Questions
Aucune question sur cette page.

---

# Page 18 — Key Exchange Mechanisms

## 📘 Résumé

L'échange de clés permet à 2 parties de convenir d'un **secret partagé** sur un canal non sûr.

| Algorithme | Principe | Points clés |
|---|---|---|
| **Diffie-Hellman (DH)** | Secret partagé sans échange préalable | Base de TLS. **Vulnérable au MITM** sans authentification. Plus lent qu'ECDH |
| **RSA** | Multiplication facile de grands nombres premiers, factorisation difficile | Signature, chiffrement, TLS, PKINIT (Kerberos). Plus lourd qu'ECC |
| **ECDH** | DH sur **courbes elliptiques** | Plus rapide et sûr. Offre la **forward secrecy**. Utilisé dans TLS et IKE |
| **ECDSA** | **Signature** numérique sur courbes elliptiques | Authentifie les parties de l'échange |

**Forward secrecy** = les communications passées restent protégées même si les clés privées sont compromises plus tard.

### IKE (Internet Key Exchange)
Protocole qui établit et maintient les sessions sécurisées d'un VPN (DH + autres techniques). Travaille souvent avec RSA et AES.

| Mode | Caractéristiques |
|---|---|
| **Main Mode** | Mode par défaut, plus **sûr** (protège l'identité), plus lent |
| **Aggressive Mode** | Moins d'échanges, plus **rapide**, **sans protection d'identité** (moins sûr) |

### Pre-Shared Key (PSK)
Secret partagé à l'avance, utilisé pour **authentifier** les parties et dériver un secret. Doit être échangé par un canal **hors bande** sûr. Si la PSK est compromise, la session l'est aussi.

> ⚠️ **Nuance** : le cours décrit Main Mode en 3 « phases » et Aggressive en 2. En réalité, ce sont des **échanges de messages** (6 messages en Main Mode, 3 en Aggressive) de la **phase 1** IKE.

## 🔑 À retenir
- DH sans authentification = MITM possible.
- ECDH = forward secrecy.
- Aggressive Mode = pas de protection d'identité (PSK capturable, d'où l'intérêt côté pentest).

## ✅ Questions
Aucune question sur cette page.

---

# Page 19 — Authentication Protocols

## 📘 Résumé

Les protocoles d'authentification vérifient l'identité des utilisateurs/équipements et protègent la confidentialité et l'intégrité des échanges.

| Protocole | Description |
|---|---|
| **Kerberos** | Authentification par **tickets** via un **KDC** (domaines) |
| **SRP** | Mot de passe + cryptographie, résiste à l'écoute et au MITM |
| **SSL / TLS** | Communication sécurisée (TLS = successeur de SSL) |
| **OAuth** | **Autorisation** déléguée sans partager le mot de passe |
| **OpenID** | Identité unique pour plusieurs sites |
| **SAML** | Échange XML d'authentification/autorisation |
| **2FA / MFA** | 2 facteurs / plusieurs facteurs (ce que je sais, ai, suis) |
| **FIDO** | Standards ouverts d'authentification forte |
| **PKI** | Infrastructure de clés publiques/privées, signatures |
| **SSO** | Un seul identifiant pour plusieurs applications |
| **PAP** | Mot de passe envoyé **en clair** |
| **CHAP** | Vérification par **three-way handshake** |
| **EAP** | Cadre pour plusieurs méthodes d'authentification |
| **SSH** | Accès distant, exécution de commandes, transfert sécurisé |
| **HTTPS** | HTTP via SSL/TLS (chiffrement + authentification du serveur) |
| **LEAP** | Wi-Fi Cisco basé EAP, RC4, vulnérable (dictionnaire) |
| **PEAP** | Tunnel TLS pour Wi-Fi/filaire, certificat serveur, largement utilisé en entreprise |

- **PEAP > LEAP** : certificat serveur et MSCHAPv2 chiffré. Les deux sont désormais remplacés par **EAP-TLS**.
- SSH et HTTPS reposent sur **SSL/TLS**, supportent certificats et PKI, et protègent contre le MITM.
- LEAP/PEAP servent à authentifier des clients Wi-Fi ou des télétravailleurs en VPN.

## 🔑 À retenir
- Kerberos = tickets/KDC. PAP = clair. CHAP = three-way handshake.
- OAuth = autorisation, SAML/OpenID = identité.
- MFA = plusieurs facteurs de natures différentes.

## ✅ Questions
Aucune question sur cette page.

---

# Page 20 — TCP/UDP Connections

## 📘 Résumé

| | **TCP** | **UDP** |
|---|---|---|
| Mode | Orienté connexion | Sans connexion |
| Fiabilité | Garantit la réception, retransmission | Aucune vérification |
| Vitesse | Plus lent | Plus rapide |
| Usage | Web, e-mail | Streaming, jeux, temps réel |

### Paquet IP
**En-tête** + **payload**. Analogie : enveloppe (en-tête) et lettre (payload).

| Champ | Rôle |
|---|---|
| Version | Version d'IP |
| IHL | Taille de l'en-tête (mots de 32 bits) |
| Class of Service | Importance de la transmission |
| Total length | Longueur du paquet |
| **Identification (ID)** | Identifie les fragments (16 bits : 0-65535) |
| Flags / Fragment Offset | Fragmentation |
| **TTL** | Durée de vie |
| Protocol | TCP, UDP… |
| Checksum | Détection d'erreurs de l'en-tête |
| Source / Destination | Adresses |
| Options / Padding | Options de routage, remplissage |

### Astuce : repérer 2 IP du même hôte
Si deux IP différentes émettent des paquets dont les **IP ID se suivent** (1337, 1338, 1339, 1340…), elles appartiennent très probablement à **la même machine**.

```text
IP 10.129.1.100.5060 > 10.129.1.1.5060: SIP, length: 1329, id 1337
IP 10.129.2.200.5060 > 10.129.1.1.5060: SIP, length: 1329, id 1340
```

### ⌨️ Tracer une route
```bash
ping -c 1 -R 10.129.143.158     # Record-Route : liste les routeurs traversés
```
La sortie `RR:` donne les IP des équipements croisés (ex. `10.10.14.38 → 10.129.0.1 → 10.129.143.158 → … → 10.10.14.38`).

**Traceroute TCP** : envoi de SYN avec TTL = 1, 2, 3… Chaque routeur renvoie un **ICMP Time Exceeded** quand le TTL tombe à 0. Fin quand la cible répond **SYN/ACK** ou **RST**.
**Traceroute UDP** (Unix) : fin quand on reçoit **Destination Unreachable / Port Unreachable**.

### TCP et UDP
- **Segment TCP** : ports source/destination, **numéro de séquence**, **numéro d'acquittement**, **flags de contrôle**, **window size**, checksum, **urgent pointer**, payload. Encapsulé dans un paquet IP.
- **Datagramme UDP** : transmission directe, sans établissement de connexion.

### Blind spoofing
Attaque où l'attaquant envoie de fausses informations **sans voir les réponses** : faux ports, faux **ISN** (Initial Sequence Number). Permet de casser ou perturber des connexions, voire de les intercepter.

## 🔑 À retenir
- IP ID consécutifs depuis 2 IP = même hôte.
- Traceroute joue sur le TTL. `ping -R` = Record-Route.
- ISN = numéro de séquence initial d'une connexion TCP.

## ✅ Questions
Aucune question sur cette page.

---

# Page 21 — Cryptography

## 📘 Résumé

Le chiffrement protège la **confidentialité** et l'**intégrité** des données en transit (paiements, e-mails, données personnelles).

### Chiffrement symétrique vs asymétrique
| | **Symétrique** | **Asymétrique** |
|---|---|---|
| Clé | **La même** pour chiffrer et déchiffrer | **Paire** : publique + privée |
| Fonctionnement | Émetteur et récepteur partagent la clé | Clé publique chiffre, clé **privée** déchiffre |
| Faiblesse | **Distribution / stockage / échange** de la clé | Plus lent, calculs lourds |
| Exemples | **AES**, **DES**, 3DES | **RSA**, **PGP**, **ECC** |
| Usage | Gros volumes (disques, réseau) | Signatures, TLS, VPN, SSH, PKI, cloud |
| Avantage | Rapide | Pas d'échange secret de clé, **signatures numériques** |

### DES et 3DES
- **DES** : chiffrement par blocs de **64 bits**, clé de 64 bits dont 8 de contrôle → **56 bits effectifs**.
- **3DES** : chiffrer avec K1, déchiffrer avec K2, rechiffrer avec K3. Plus sûr que DES mais limité par la clé de 56 bits.

### AES
| Variante | Clé |
|---|---|
| AES-128 | 128 bits |
| AES-192 | 192 bits |
| AES-256 | 256 bits |

Plus rapide que DES (traite plusieurs blocs). Utilisé dans **Wi-Fi 802.11i**, **IPsec**, **SSH**, **VoIP**, **PGP**, **OpenSSL**.

### Modes de chiffrement par blocs
| Mode | Caractéristiques | Usage |
|---|---|---|
| **ECB** (Electronic Code Book) | **À éviter** : ne cache pas les motifs, analyse statistique possible | — |
| **CBC** (Cipher Block Chaining) | Chaînage des blocs | Chiffrement de disques, e-mails, TLS/SSL, VeraCrypt |
| **CFB** (Cipher Feedback) | Flux en temps réel | PKCS, BitLocker |
| **OFB** (Output Feedback) | Flux, génération de la clé de flux | PKCS, SSH |
| **CTR** (Counter) | Flux temps réel | IPsec, BitLocker |
| **GCM** (Galois/Counter) | **Confidentialité + intégrité** ensemble | Wi-Fi, VPN, protocoles sécurisés |

> ⚠️ **Nuance** : le cours dit que CBC est « le mode par défaut d'AES ». Cela dépend de la bibliothèque. À retenir surtout : **ECB = mauvais choix**, **GCM = chiffrement authentifié**.

## 🔑 À retenir
- Symétrique : 1 clé, rapide, problème d'échange. Asymétrique : 2 clés.
- DES = 56 bits effectifs. AES = 128/192/256.
- ECB déconseillé. GCM = confidentialité + intégrité.

## ✅ Questions
Aucune question sur cette page.

---

# Mémo final — toutes les réponses HTB

| Page | Question | **Réponse à mettre sur HTB** |
|---|---|---|
| 1 à 9 | *(pas de question)* | — |
| 10 | Masque décimal de `10.200.20.0/27` | `255.255.255.224` |
| 10 | Broadcast de `10.200.20.0/27` | `10.200.20.31` |
| 10 | `/27` en 4 sous-réseaux : réseau du 3e | `10.200.20.16` |
| 10 | `/27` en 4 sous-réseaux : broadcast du 2e | `10.200.20.15` |
| 11 à 21 | *(pas de question)* | — |

---

# Cheat-sheet du module

## Calculs de subnetting
| Besoin | Formule |
|---|---|
| Hôtes utilisables | `2^(32 − CIDR) − 2` |
| Taille du bloc (octet qui change) | `256 / 2^(CIDR % 8)` |
| Adresse réseau | bits hôte à 0 |
| Broadcast | bits hôte à 1 (réseau + taille − 1) |
| Découper en N sous-réseaux | ajouter `log2(N)` bits au CIDR |

## Commandes
```bash
ping -c 1 -R IP                                              # Record-Route
traceroute IP                                                # route (UDP par défaut sous Unix)
sudo modprobe 8021q                                          # module VLAN Linux
sudo ip link add link eth0 name eth0.20 type vlan id 20      # crée un VLAN
sudo ip addr add 192.168.1.1/24 dev eth0.20                  # IP sur le VLAN
sudo ip link set up eth0.20                                  # active
tshark -r capture.pcapng -T fields -e vlan.id | sort -n -u   # VLAN présents dans une capture
```
```powershell
Get-NetAdapter | Format-Table -AutoSize
Get-NetAdapterAdvancedProperty -DisplayName "vlan id"
Set-NetAdapter -Name "Ethernet 2" -VlanID 10
```

## Filtres Wireshark
| Filtre | Effet |
|---|---|
| `vlan` | Trames 802.1Q |
| `vlan.id == 10` | VLAN 10 |

## Ports essentiels
| Port | Service |
|---|---|
| 21 | FTP |
| 22 | SSH |
| 23 | Telnet |
| 25 | SMTP |
| 53 | DNS |
| 67/68 | DHCP |
| 80 / 443 | HTTP / HTTPS |
| 88 | Kerberos |
| 110 / 143 | POP3 / IMAP |
| 123 | NTP |
| 161 | SNMP |
| 389 | LDAP |
| 445 | SMB |
| 500 | IKE / ISAKMP |
| 1723 | PPTP |
| 3306 | MySQL |
| 3389 | RDP |
| 5060 | SIP |

## Repères rapides
| Sujet | À retenir |
|---|---|
| TTL | 128 = Windows, 64 = Linux/macOS, 255 = Solaris |
| RFC 1918 | `10/8`, `172.16/12`, `192.168/16` |
| MAC | 48 bits, OUI 3 octets + NIC 3 octets |
| IPv6 | 128 bits, pas de broadcast, `::` une seule fois |
| 802.1Q | TPID `0x8100`, VID 12 bits, 4094 VLAN |
| VXLAN | VNI 24 bits (~16 millions) |
| IPsec | AH = protocole **51**, ESP = protocole **50**, IKE UDP/500, NAT-T UDP/4500 |
| WEP | RC4, IV 24 bits, cassable |
| ECB | À éviter. GCM = confidentialité + intégrité |
