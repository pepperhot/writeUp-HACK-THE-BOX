
# HTB — [NOM DU LAB]

> **Difficulté :** [Very Easy / Easy / Medium / Hard]  
> **OS :** [Linux / Windows]  
> **IP :** `[IP]`  
> **Date :** [DATE]

---

## 1. Objectif

Quelques lignes pour expliquer ce que l'on doit faire sur cette machine et, si elle est connue, quelle vulnérabilité ou technique principale est utilisée.

**Objectif :**  
[Expliquer rapidement le but du lab]

---

## 2. Reconnaissance

### Scan Nmap

```bash
nmap -sV [IP]
```

**Résultat :**

```text
[Résultat intéressant du scan]
```

**Ce que l'on remarque :**

- Port `[XX]` → [service]
    
- Port `[XX]` → [service]
    
- [Autre information intéressante]
    

---

## 3. Énumération

### [Service / Site découvert]

**Commande utilisée :**

```bash
[commande]
```

**Résultat :**

```text
[résultat]
```

**Interprétation :**

[Expliquer simplement ce que cette découverte nous apprend et pourquoi on continue dans cette direction.]

### Autres découvertes

- `[chemin / endpoint / fichier]` → [explication]
    
- `[chemin / endpoint / fichier]` → [explication]
    

---

## 4. Vulnérabilité

### Identification

**Vulnérabilité :** [Nom / CVE / mauvaise configuration]

**Explication :**

[Expliquer simplement pourquoi la cible est vulnérable.]

**Comment on l'a trouvée :**

[Expliquer le cheminement : scan → service → version → recherche → test...]

---

## 5. Exploitation

### Étape 1 — [Nom de l'étape]

**Requête / commande :**

```bash
[commande]
```

ou

```http
[requête HTTP]
```

**Résultat :**

```text
[résultat]
```

**Explication :**

[Qu'est-ce qui s'est passé ? Pourquoi cette réponse est intéressante ?]

---

### Étape 2 — [Nom de l'étape]

**Commande :**

```bash
[commande]
```

**Résultat :**

```text
[résultat]
```

**Explication :**

[Explication simple]

---

## 6. Accès obtenu

**Accès :** [Shell / RCE / compte utilisateur / root / Administrator...]

**Vérification :**

```bash
[commande]
```

**Résultat :**

```text
[résultat]
```

**Interprétation :**

[Expliquer les droits obtenus.]

---

## 7. Récupération des flags

### User Flag

```bash
[commande]
```

```text
[flag]
```

### Root Flag

```bash
[commande]
```

```text
[flag]
```

---

## 8. Chemin d'exploitation

Résumé du chemin suivi pendant le lab :

```text
Reconnaissance
      ↓
Nmap
      ↓
[Service découvert]
      ↓
[Énumération]
      ↓
[Vulnérabilité]
      ↓
[Exploitation]
      ↓
[Accès]
      ↓
User Flag / Root Flag
```

---

## 9. Résumé

**Machine :** [Nom]

**Vulnérabilité principale :** [Vulnérabilité]

**Accès initial :** [Accès]

**Privilèges obtenus :** [Utilisateur / Root / Administrator]

**Flags :**

- User : `[flag]`
    
- Root : `[flag]`
    

**Méthode en une phrase :**  
[Résumer l'exploitation en une seule phrase.]