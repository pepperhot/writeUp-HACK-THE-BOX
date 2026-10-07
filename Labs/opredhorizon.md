# Writeup — OpRedHorizon : Cosmic Corruption (HTB Sherlock, DFIR, very easy)

> **Scénario** : Circuitbreaker a détecté un uplink suspect : un fichier a été mis en file de diffusion du satellite via **CFDP** (CCSDS File Delivery Protocol). On doit reconstruire le fichier et comprendre ce qu'il contient.

- **Archive** : `opredhorizon_cosmiccorruption.zip` — mot de passe `hacktheblue`
- **Contenu** : `CosmicCorruption.pcap` (≈ 10,8 Mo, 9925 paquets)

---

## Réponses

| # | Question | Réponse |
|---|---|---|
| 1 | What protocol is being used in the UDP data? | `CFDP` |
| 2 | How many bytes are in the file data in each payload? | `1024` |
| 3 | What is the file type of the transferred file? | `WAV` |
| 4 | What transmission/encoding method was used to embed an image in the file? | `SSTV` |
| 5 | What is the comment in the transferred file? | `SilentCrown` |
| 6 | What is the message that can be seen alongside the code? | `Scan QR Code for Free Crypto` ⚠️ |
| 7 | What is the decoded value? | `https://phoenix-continuity.zenium-gs.htb/Truth` |

⚠️ **Q6** : l'image SSTV est en basse résolution et on y lit « Scan QRCode… » (sans espace). Cette version a été **refusée** par la plateforme. La version avec espace `Scan QR Code for Free Crypto` est la forme attendue la plus probable (non confirmée au moment de la rédaction). Variantes à tester si besoin : point final, espaces parasites.

---

## Prérequis

```bash
# Linux (Debian/Ubuntu/Kali)
sudo apt install unzip tshark zbar-tools git python3-pip
pip install scapy numpy scipy pillow soundfile
```

Mise en place :

```bash
mkdir cosmic && cd cosmic
unzip -P hacktheblue opredhorizon_cosmiccorruption.zip
# -> CosmicCorruption.pcap
```

---

## Étape 1 — Reconnaissance du trafic (Q1)

Ouvrir le pcap dans Wireshark ou faire un premier tri en ligne de commande :

```bash
tshark -r CosmicCorruption.pcap -q -z conv,udp
```

On observe **un seul flux UDP** : `10.7.16.11:58665 → 10.7.16.69:5111`.

Répartition des tailles de payload UDP :

| Taille payload | Nombre | Rôle |
|---|---|---|
| 63 octets | 1 | 1er paquet → Metadata PDU |
| 1035 octets | 9922 | File Data PDU |
| 685 octets | 1 | dernier bloc de données (fin de fichier) |
| 18 octets | 1 | EOF PDU |

Le premier paquet contient en clair deux chemins de fichiers :

```
/onboard/Authority.wav  ->  /onboard/store/Message.wav
```

Ports non standards + structure « métadonnées → blocs de données à offset → EOF » + contexte satellite = **CFDP** (CCSDS 727.0-B), le protocole de transfert de fichiers spatial (NASA/ESA).

> 💡 Wireshark ne décode pas forcément ce port automatiquement : *clic droit → Decode As… → UDP port 5111 → CFDP* si le dissecteur est disponible dans ta version.

**✅ Q1 : `CFDP`**

---

## Étape 2 — Disséquer l'en-tête CFDP (Q2)

Premiers octets d'un paquet de 1035 octets :

```
34 04 04 00 01 00 02 | 00 00 04 00 | 1b a3 0a be ...
└──── en-tête (7) ───┘ └ offset (4) ┘ └── données ──
```

### En-tête fixe CFDP (4 octets)

| Octet(s) | Valeur | Signification |
|---|---|---|
| 0 | `0x34` = `0011 0100` | version=1, **PDU type=1 (File Data)**, direction=0, mode=1 (unacknowledged) |
| 1-2 | `0x0404` = **1028** | longueur du champ de données du PDU |
| 3 | `0x00` | taille Entity ID = 0+1 = **1 octet**, taille Seq Num = 0+1 = **1 octet** |

### Champs variables (3 octets)

| Octet | Valeur | Champ |
|---|---|---|
| 4 | `0x01` | Source Entity ID |
| 5 | `0x00` | Transaction Sequence Number |
| 6 | `0x02` | Destination Entity ID |

→ en-tête total = **7 octets**.

### Corps d'un File Data PDU

- 4 octets : **offset** du bloc dans le fichier (big-endian)
- le reste : données du fichier

Calcul : `1028 (longueur) − 4 (offset) = 1024`.
Vérification : `7 + 4 + 1024 = 1035` ✔ (taille du payload UDP).

On voit aussi l'offset progresser de `0x400` (1024) en `0x400` d'un paquet à l'autre.

**✅ Q2 : `1024`**

---

## Étape 3 — Le Metadata PDU (Q3)

Le paquet de 63 octets commence par `0x24` → bit PDU type = 0 → **File Directive**.

```
24 00 38 00 01 00 02 | 07 | 43 | 00 9b 0a a2 | 16 "/onboard/Authority.wav" | 1a "/onboard/store/Message.wav"
     en-tête (7)       code flags  taille fichier   len + nom source            len + nom destination
```

| Champ | Valeur |
|---|---|
| Directive code | `0x07` = Metadata |
| Taille du fichier | `0x009b0aa2` = **10 160 802 octets** |
| Fichier source | `/onboard/Authority.wav` |
| Fichier destination | `/onboard/store/Message.wav` |

Le paquet de 18 octets a le directive code `0x04` = **EOF** (contient checksum + taille).

**✅ Q3 : `WAV`** (confirmé après reconstruction par `file` : *RIFF WAVE, Microsoft PCM, 16 bit, mono 44100 Hz*)

---

## Étape 4 — Reconstruire le fichier

Script `cfdp_extract.py` :

```python
#!/usr/bin/env python3
"""Reconstruit un fichier transféré en CFDP (CCSDS 727.0) depuis un pcap.
Usage : python3 cfdp_extract.py CosmicCorruption.pcap Message.wav
"""
import sys
from scapy.all import rdpcap, UDP

pcap, out = sys.argv[1], sys.argv[2]
chunks = {}

for pkt in rdpcap(pcap):
    if UDP not in pkt:
        continue
    pdu = bytes(pkt[UDP].payload)

    # --- En-tête CFDP fixe (4 octets) ---
    flags = pdu[0]
    pdu_type = (flags >> 4) & 1           # 0 = File Directive, 1 = File Data
    lens = pdu[3]
    eid_len = ((lens >> 4) & 0x7) + 1     # taille des Entity ID
    seq_len = (lens & 0x7) + 1            # taille du Transaction Seq Num
    hdr = 4 + eid_len + seq_len + eid_len # src + seq + dst -> ici 7 octets
    body = pdu[hdr:]

    if pdu_type == 1:                     # File Data PDU
        offset = int.from_bytes(body[:4], "big")
        chunks[offset] = body[4:]
    elif body and body[0] == 0x07:        # Metadata PDU
        size = int.from_bytes(body[2:6], "big")
        n = body[6]
        src = body[7:7 + n].decode()
        dst = body[8 + n:8 + n + body[7 + n]].decode()
        print(f"[meta] {src} -> {dst} ({size} octets)")
    elif body and body[0] == 0x04:        # EOF PDU
        print("[eof] fin de transfert")

data = b"".join(chunks[o] for o in sorted(chunks))
open(out, "wb").write(data)
print(f"[ok] {len(chunks)} blocs, {len(data)} octets -> {out}")
```

Exécution :

```bash
python3 cfdp_extract.py CosmicCorruption.pcap Message.wav
file Message.wav
```

Sortie attendue :

```
[meta] /onboard/Authority.wav -> /onboard/store/Message.wav (10160802 octets)
[eof] fin de transfert
[ok] 9923 blocs, 10160802 octets -> Message.wav
Message.wav: RIFF (little-endian) data, WAVE audio, Microsoft PCM, 16 bit, mono 44100 Hz
```

### ⚠️ Pièges

- **Ne pas filtrer uniquement sur les paquets de 1035 octets** : le dernier bloc (offset `0x9b0800`) ne fait que 685 octets. Sans lui, le fichier est tronqué (10 160 128 au lieu de 10 160 802 octets).
- **Toujours trier par offset**, pas par ordre d'arrivée : UDP ne garantit pas l'ordre.
- La taille finale doit correspondre à celle annoncée dans le Metadata PDU (`10160802`).

### Alternative sans Scapy

```bash
tshark -r CosmicCorruption.pcap -Y udp -T fields -e udp.payload > payloads.hex
```

Puis, pour chaque ligne : ignorer les 7 premiers octets, lire l'offset sur 4 octets, écrire le reste à cet offset (Python ou CyberChef).

---

## Étape 5 — Métadonnées du WAV (Q5)

Un WAV (RIFF) peut contenir un chunk `LIST/INFO` avec des tags texte.

```bash
xxd Message.wav | head -8
# ou
strings -n 6 Message.wav | head
# ou
exiftool Message.wav
```

En-tête observé :

```
RIFF....WAVEfmt ........LIST`...INFO
IART  NotVorkane
ICMT  SilentCrown
INAM  Zenium-SAT-06 BEACON
ISFT  Lavf62.3.100
```

| Tag RIFF | Signification | Valeur |
|---|---|---|
| `IART` | Artiste | `NotVorkane` |
| `ICMT` | **Commentaire** | **`SilentCrown`** |
| `INAM` | Titre | `Zenium-SAT-06 BEACON` |
| `ISFT` | Logiciel | `Lavf62.3.100` (FFmpeg) |

**✅ Q5 : `SilentCrown`**

---

## Étape 6 — Identifier l'encodage de l'image (Q4)

Le fichier est un « beacon » audio de 10 Mo (~115 s à 44,1 kHz). Dans **Audacity** ou **Sonic Visualiser**, la vue *spectrogramme* montre :

- des impulsions de synchronisation régulières à **1200 Hz** ;
- un signal balayant **1500–2300 Hz** (luminance des pixels).

C'est la signature de la **SSTV** (Slow-Scan Television), technique radioamateur pour transmettre une image par audio (utilisée notamment par l'ISS).

**✅ Q4 : `SSTV`**

---

## Étape 7 — Décoder l'image SSTV (Q6)

### Option A — `colaclanth/sstv` (CLI)

> ⚠️ Le paquet `sstv` sur PyPI est un **autre projet**. Installer depuis GitHub.

```bash
git clone https://github.com/colaclanth/sstv
cd sstv
python3 -m sstv -d ../Message.wav -o ../sstv.png
cd ..
```

Sortie :

```
[sstv] Searching for calibration header... Found!
[sstv] Detected SSTV mode Martin 1
[sstv] Decoding image... 100%
[sstv] ...Done!
```

Si erreur `OSError: [Errno 25] Inappropriate ioctl for device` (pas de vrai terminal, ex. lancé depuis un script/IDE) :

```bash
sed -i 's/cols = get_terminal_size().columns/cols = 80/' sstv/common.py
```

### Option B — outils graphiques

- **QSSTV** (Linux), **RX-SSTV** / **MMSSTV** (Windows) : lire le WAV en entrée.
- **Robot36** (Android) : jouer le WAV près du micro du téléphone.

### Résultat

Image 320×256 (mode **Martin 1**) : un **QR code** avec la légende :

```
Scan QR Code for Free Crypto
```

(voir l'avertissement Q6 en haut : la résolution rend l'espace entre « QR » et « Code » ambigu ; « QRCode » collé a été refusé).

**✅ Q6 : `Scan QR Code for Free Crypto`**

---

## Étape 8 — Décoder le QR code (Q7)

```bash
zbarimg -q sstv.png
```

ou en Python :

```bash
pip install opencv-python
python3 -c "import cv2; print(cv2.QRCodeDetector().detectAndDecode(cv2.imread('sstv.png'))[0])"
```

Résultat :

```
https://phoenix-continuity.zenium-gs.htb/Truth
```

**✅ Q7 : `https://phoenix-continuity.zenium-gs.htb/Truth`**

---

## Récapitulatif de la chaîne

```
pcap ──(UDP 5111)──► PDU CFDP ──(tri par offset)──► Message.wav
                                                      │
                          ┌───────────────────────────┤
                          ▼                           ▼
                 LIST/INFO : ICMT=SilentCrown   audio SSTV (Martin 1)
                                                      │
                                                      ▼
                                   image : "Scan QR Code for Free Crypto" + QR
                                                      │
                                                      ▼
                              https://phoenix-continuity.zenium-gs.htb/Truth
```

## IOC / analyse DFIR

| Type | Valeur |
|---|---|
| IP source (uplink) | `10.7.16.11` (port 58665) |
| IP destination | `10.7.16.69` (port UDP 5111) |
| Entités CFDP | source `1` → destination `2` |
| Fichier source | `/onboard/Authority.wav` |
| Fichier déposé | `/onboard/store/Message.wav` (10 160 802 octets) |
| Tags WAV | `IART=NotVorkane`, `ICMT=SilentCrown`, `INAM=Zenium-SAT-06 BEACON` |
| URL (quishing) | `https://phoenix-continuity.zenium-gs.htb/Truth` |

**Lecture de l'incident** : un opérateur a injecté via CFDP un faux « beacon » dans la file de diffusion du satellite. Le WAV porte une image SSTV de **quishing** (phishing par QR code, appât « Free Crypto ») destinée à tous les récepteurs au sol qui décodent le beacon. CFDP n'offre ni authentification ni chiffrement natifs : quiconque accède à l'uplink peut injecter un fichier (contre-mesure : CCSDS SDLS, liste blanche des entités source et des chemins de destination).

## Outils utilisés

| Étape | Outil |
|---|---|
| Analyse réseau | Wireshark / tshark |
| Reconstruction CFDP | Python + Scapy |
| Métadonnées WAV | xxd / strings / exiftool |
| Spectrogramme | Audacity / Sonic Visualiser |
| Décodage SSTV | colaclanth/sstv, QSSTV, RX-SSTV, Robot36 |
| QR code | zbarimg / OpenCV |