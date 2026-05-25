# lab8-SEC
# 🔐 Lab — Analyse de Sécurité Mobile

> **Cadre pédagogique** · Analyse statique d'APK · Blue Team / SOC  
> Durée estimée : ~2h · Outils : BeVigil · Yaazhini · OWASP MASVS

---

## 📁 Structure du workspace

```
lab-mobile-security/
├── 00-scope/
│   ├── scope.md            ← Périmètre, autorisation, limites éthiques
│   └── targets.txt         ← (Option B) Domaines autorisés
├── 01-bevigil/
│   └── bevigil_notes.md    ← Notes structurées de l'analyse BeVigil
├── 02-yaazhini/
│   └── yaazhini_notes.md   ← Notes structurées du rapport Yaazhini
├── 03-triage/
│   ├── triage.csv          ← Consolidation des constats (min. 10 lignes)
│   └── owasp_mapping.md    ← Corrélation avec OWASP MASVS (min. 5 constats)
├── 04-report/
│   └── rapport_final.md    ← Rapport de synthèse actionnable
├── analyse_info.txt         ← Métadonnées de l'analyse
├── commands.log             ← Traçabilité de toutes les commandes
└── checklist_fin.md         ← Checklist de clôture signée
```

---

## 🗺️ Déroulement des tâches

| # | Tâche | Durée | Livrable |
|---|-------|-------|----------|
| 0 | Règles, périmètre et éthique | 5 min | `00-scope/scope.md` |
| 1 | Préparation du workspace et traçabilité | 10 min | `analyse_info.txt`, `commands.log` |
| 2 | Préparer l'artefact autorisé | 10 min | APK + hash SHA-256 ou `targets.txt` |
| 4 | Collecte BeVigil | 20 min | `01-bevigil/bevigil_notes.md` |
| 5 | Démarrage Yaazhini | 15 min | Rapport Yaazhini dans `02-yaazhini/` |
| 6 | Collecte Yaazhini — indices d'exposition | 20 min | `02-yaazhini/yaazhini_notes.md` |
| 7 | Normalisation et dédoublonnage | 15 min | `03-triage/triage.csv` |
| 8 | Corrélation OWASP | 15 min | `03-triage/owasp_mapping.md` |
| 9 | Rédaction du mini-rapport | 20 min | `04-report/rapport_final.md` |
| 10 | Nettoyage et clôture | 5 min | `checklist_fin.md` |

---

## ⚙️ Initialisation rapide (PowerShell)

```powershell
# Créer toute l'arborescence en une seule passe
mkdir 00-scope, 01-bevigil, 02-yaazhini, 03-triage, 04-report

# Initialiser le fichier de traçabilité
"# Log des commandes - $(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')" |
  Out-File -FilePath "commands.log" -Encoding utf8

# Initialiser les métadonnées
@"
Date: $(Get-Date -Format "yyyy-MM-dd")
Analyste: [Prénom Nom]
Cible: [Nom de l'application]
Artefact: [Nom du fichier APK ou liste des domaines]
Provenance: [Enseignant / Projet de cours]
Hash: [À remplir si APK]
Versions outils:
  - BeVigil: [version]
  - Yaazhini: [version]
"@ | Out-File -FilePath "analyse_info.txt" -Encoding utf8
```

---

## 🔒 Tâche 0 — Périmètre et éthique

Créer `00-scope/scope.md` **avant toute action** :

```powershell
@"
# Périmètre d'analyse

## Cible autorisée
Nom: [Nom de l'application ou du domaine]
Propriétaire: [Propriétaire]

## Autorisation
Source: [Référence au cours/TP]
Preuve: [Email de l'enseignant / Document d'autorisation]

## Type d'artefact
- [x] APK pédagogique fourni par l'enseignant
- [ ] Application interne autorisée
- [ ] Domaine explicitement autorisé

## Limites explicites
- Pas d'exploitation des vulnérabilités découvertes
- Pas de tests intrusifs
- Pas de contournement des mécanismes de sécurité

## Période d'analyse
Date de début: $(Get-Date -Format "yyyy-MM-dd")
Durée prévue: 2 heures
"@ | Out-File -FilePath "00-scope\scope.md" -Encoding utf8
```

> ⚠️ **Point de vigilance** : L'absence de périmètre documenté peut entraîner des dérives hors cadre légal. Ce fichier sert de preuve en cas de questionnement.

---

## 🧮 Tâche 2 — Artefact autorisé

### Option A — APK pédagogique

```powershell
# Copier l'APK
Copy-Item -Path "[chemin]\application_pedagogique.apk" -Destination "00-scope\"

# Calculer et injecter le hash SHA-256
$hash = Get-FileHash -Path "00-scope\application_pedagogique.apk" -Algorithm SHA256
(Get-Content "analyse_info.txt") -replace "Hash: \[À remplir si APK\]", "Hash: $($hash.Hash)" |
  Set-Content "analyse_info.txt"
```

### Option B — Domaine autorisé

```powershell
@"
example.com
api.example.com
mobile.example.com
"@ | Out-File -FilePath "00-scope\targets.txt" -Encoding utf8
```

---

## 🌐 Tâche 4 — Collecte BeVigil

Créer `01-bevigil/bevigil_notes.md` en documentant :

| Catégorie | Éléments à chercher |
|-----------|---------------------|
| Domaines & sous-domaines | Sections "Domains", "Subdomains" |
| Endpoints & APIs | Patterns `/api/v1/`, `/rest/` |
| URLs HTTP vs HTTPS | Données en transit non chiffrées |
| Emails & identifiants | Code source, métadonnées |
| Technologies & versions | Frameworks obsolètes avec CVE connus |

Structure des notes :
```markdown
## Ce qui est certain
## Ce qui est hypothèse
## Points d'intérêt
## Domaines et sous-domaines
## Endpoints et APIs
## Emails et identifiants
## Technologies détectées
```

> 💡 Distinguer **faits avérés** et **hypothèses** est essentiel pour éviter les faux positifs.

---

## 🔬 Tâche 5 & 6 — Yaazhini

### Lancement (exécutable)
```powershell
& "[chemin_yaazhini]\yaazhini.exe" -apk "00-scope\application_pedagogique.apk" -output "02-yaazhini"
```

### Lancement (script Python)
```powershell
python "[chemin_yaazhini]\yaazhini.py" --apk "00-scope\application_pedagogique.apk" --output "02-yaazhini"
```

### Éléments à documenter dans `yaazhini_notes.md`

Pour chaque élément identifié (min. 5) :

```markdown
### Élément N: [Nom/Type]
- **Localisation**: [Chemin/Section dans le rapport]
- **Description**: [Description]
- **Impact potentiel**: [Hypothèse d'impact]
- **Remédiation suggérée**: [Suggestion]
```

Catégories prioritaires à investiguer :

| Type | Indicateur dans le rapport |
|------|---------------------------|
| Secrets / API Keys | `api_key`, `password`, `token`, `secret` |
| Endpoints hardcodés | URLs complètes dans le code |
| Config sensible | `debuggable="true"`, `allowBackup="true"`, `usesCleartextTraffic` |
| Permissions excessives | `CAMERA`, `LOCATION`, `CONTACTS`, `READ_SMS` |
| Composants exposés | `exported="true"` sans permission associée |

> 🔴 **Règle absolue** : Ne jamais copier la valeur réelle d'un secret. Remplacer par `[MASQUÉ]`.

---

## 📊 Tâche 7 — Triage CSV

Colonnes du fichier `03-triage/triage.csv` :

```
ID | Source | Élément | Preuve | Confiance | Sévérité | Impact | Recommandation | Référence OWASP | Statut
```

Valeurs attendues :
- **Source** : `BeVigil` / `Yaazhini` / `BeVigil+Yaazhini`
- **Confiance** : `Faible` / `Moyenne` / `Forte`
- **Sévérité** : `Info` / `Low` / `Medium` / `High`
- **Statut** : `À confirmer` / `Confirmé` / `Faux positif`

Exemple de ligne :
```
FIND-001,BeVigil,API Key exposée,bevigil_export.json:section assets,Forte,High,Accès non autorisé aux services backend,Utiliser un stockage sécurisé des clés,MASVS-STORAGE-1,Confirmé
```

---

## 🗂️ Tâche 8 — Mapping OWASP MASVS

Référence : [github.com/OWASP/owasp-masvs](https://github.com/OWASP/owasp-masvs)

| Catégorie MASVS | Domaine couvert |
|-----------------|-----------------|
| `MASVS-STORAGE` | Stockage des données sensibles |
| `MASVS-CRYPTO` | Pratiques cryptographiques |
| `MASVS-AUTH` | Authentification et gestion de session |
| `MASVS-NETWORK` | Communications réseau |
| `MASVS-PLATFORM` | Interactions avec la plateforme Android/iOS |
| `MASVS-CODE` | Qualité et sécurité du code |
| `MASVS-RESILIENCE` | Résistance aux attaques runtime |

Structure de `owasp_mapping.md` pour chaque constat :
```markdown
## FIND-00X: [Description courte]
- **Catégorie OWASP**: MASVS-XXX
- **Référence spécifique**: VX.X
- **Justification**: [Explication en une phrase]
```

---

## 📝 Tâche 9 — Rapport final

Structure de `04-report/rapport_final.md` :

```
A. Informations générales
B. Résumé exécutif (5 lignes max)
C. Top 5 constats (sévérité + preuve + impact + remédiation + référence OWASP)
D. Faux positifs notables
E. Recommandations prioritaires (3 actions actionnables)
F. Annexes (liens vers exports)
```

> 💡 Prioriser les constats : **High → Medium → Low → Info**

---

## ✅ Tâche 10 — Checklist de clôture

```powershell
@"
# Checklist de fin d'analyse

## Périmètre et traçabilité
- [ ] Scope clairement défini et respecté
- [ ] Hash de l'APK documenté
- [ ] commands.log complet

## Collecte et analyse
- [ ] Exports BeVigil sauvegardés
- [ ] Rapport Yaazhini sauvegardé
- [ ] bevigil_notes.md et yaazhini_notes.md complètes

## Triage et reporting
- [ ] triage.csv : min. 10 lignes
- [ ] owasp_mapping.md : min. 5 constats mappés
- [ ] rapport_final.md complet

## Sécurité et conformité
- [ ] Aucun secret exposé (vérifier : password, key, token, secret)
- [ ] Aucune donnée personnelle exposée
- [ ] Aucune technique d'exploitation documentée

Je soussigné(e) [Prénom Nom] certifie avoir réalisé cette analyse dans le respect
du périmètre autorisé et des règles éthiques définies.

Date: $(Get-Date -Format "yyyy-MM-dd")
Signature: [Prénom Nom]
"@ | Out-File -FilePath "checklist_fin.md" -Encoding utf8
```

---

## 🛡️ Rappels éthiques

- **Aucune exploitation** des vulnérabilités identifiées
- **Aucun test intrusif** hors périmètre autorisé
- **Masquer systématiquement** toute valeur sensible trouvée (`[MASQUÉ]`)
- **Documenter** chaque action dans `commands.log`
- **Analyser uniquement** les artefacts explicitement autorisés

---

*Lab réalisé dans un cadre pédagogique — Analyse statique uniquement*
