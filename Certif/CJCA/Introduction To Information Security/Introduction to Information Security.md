# CJCA — Structure of InfoSec

## 1. Bases

**InfoSec** = protéger les informations et systèmes contre les accès, modifications ou destructions non autorisés.

### À ne pas confondre

* **Vulnérabilité** → faiblesse d'un système
* **Menace** → ce qui peut exploiter la faiblesse
* **Risque** → dommage potentiel = probabilité × impact

---

## 2. CIA Triad

| Concept | Signification   | Exemple     |
| ------- | --------------- | ----------- |
| **C**   | Confidentiality | Chiffrement |
| **I**   | Integrity       | Hachage     |
| **A**   | Availability    | Redondance  |

Autres principes : **Authentication**, **Non-repudiation**, **Privacy**.

---

## 3. Les 8 domaines

1. **Network Security** → réseau, firewall, VPN
2. **Application Security** → SQLi, XSS, Security by Design
3. **Operational Security** → assets, menaces, vulnérabilités, monitoring
4. **DR & BC** → récupération et continuité d'activité
5. **Cloud Security** → responsabilité partagée, IAM
6. **Physical Security** → protection des locaux et matériels
7. **Mobile Security** → Device, Data, Network, Application
8. **IoT Security** → segmentation réseau

### DR / BC

* **DR** → restaurer les systèmes
* **BC** → maintenir l'activité

---

## 4. Menaces principales

* **DDoS** → surcharge par trafic massif → *Mirai / Dyn*
* **Ransomware** → chiffrement + rançon → *WannaCry*
* **Social Engineering** → manipulation humaine
* **Insider Threat** → abus d'un accès légitime
* **APT** → attaque longue et discrète → *SolarWinds*

### Social Engineering

**Phishing / Pretexting / Baiting / Tailgating / Quid Pro Quo**

---

## 5. Équipes

* 🔴 **Red Team** → simule l'attaquant
* 🔵 **Blue Team** → défend et surveille
* 🟣 **Purple Team** → Red + Blue

### SOC

* **Tier 1** → triage
* **Tier 2** → investigations
* **Tier 3** → incidents critiques

---

## 6. Métiers

* **CISO** → dirige la sécurité
* **Security Architect** → conçoit les systèmes sécurisés
* **Pentester** → recherche les vulnérabilités
* **Incident Responder** → gère les incidents
* **Security Analyst** → analyse les alertes
* **Compliance Specialist** → normes/réglementations
* **Bug Bounty Hunter** → recherche de vulnérabilités

---

## 7. Incident Response

```text
Investigation
→ Confinement
→ Éradication
→ Récupération
→ Retour d'expérience
```

---

## 8. Processus InfoSec

```text
Risk Assessment
→ Security Planning
→ Security Controls
→ Monitoring & Detection
→ Incident Response
→ Disaster Recovery
→ Continuous Improvement
```

---

## 9. Outils à connaître

| Outil               | Rôle                     |
| ------------------- | ------------------------ |
| **Firewall**        | Filtre le trafic         |
| **IDS**             | Détecte                  |
| **IPS**             | Bloque                   |
| **SIEM**            | Analyse les événements   |
| **Nmap**            | Scan réseau              |
| **Wireshark**       | Analyse réseau           |
| **Metasploit**      | Exploitation             |
| **Burp Suite**      | Sécurité Web             |
| **John the Ripper** | Cassage de mots de passe |

---

## 10. Questions importantes

* **C de CIA** → Confidentiality
* **DR** → Disaster Recovery
* **Sécurité mobile** → 4 couches
* **CISO** → Chief Information Security Officer

### ⚠️ À retenir

> **Menace ≠ Vulnérabilité ≠ Risque**
> **IDS détecte / IPS bloque**
> **DR restaure / BC maintient l'activité**
> **Red attaque / Blue défend / Purple collabore**
> **Toujours avoir une autorisation avant un test de sécurité.**
