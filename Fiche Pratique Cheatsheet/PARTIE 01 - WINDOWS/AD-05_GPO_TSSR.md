# AD-05 — GPO (Group Policy Objects)

> Collections virtuelles de politiques de sécurité pour gérer un parc Windows de manière centralisée.  
> **CCP2** (Exploiter serveurs Windows/AD) + **CCP5** (Infrastructure virtualisée).

---

## Définition & composantes

| Composante | Emplacement | Rôle |
|-----------|-------------|------|
| **Entrée LDAP** | `CN=Policies,CN=System,DC=dom,DC=lan` | Métadonnées (nom, GUID, droits) |
| **Contenu SYSVOL** | `\\dom\SYSVOL\dom\Policies\{GUID}\` | Fichiers de configuration |
| **Attribut gPLink** | Sur l'objet OU/Site/Domaine | Lien GPO ↔ conteneur AD |

---

## Ce que peut gérer une GPO

| Catégorie | Exemples |
|-----------|----------|
| **Sécurité** | Politique de mots de passe, verrouillage de compte |
| **Interface** | Fond d'écran, menu Démarrer, restrictions |
| **Déploiement** | Scripts logon/logoff, installation logiciels |
| **Réseau** | Montage de lecteurs, imprimantes |
| **Redirection** | Dossiers Documents, Bureau → partage réseau |

> GPO **Windows uniquement** — ignorées par les clients Linux.

---

## Priorité LSDOU

> Ordre d'application (du plus faible au plus fort) :

```
L → Local       (stratégies locales - gpedit.msc)
S → Site        (GPO de site AD)
D → Domain      (GPO de domaine)
O → OU          (GPO d'Unité d'Organisation) ← priorité la plus FORTE
```

> Plus on descend dans LSDOU, plus la GPO est prioritaire.  
> Sur une même OU : GPO en **bas de liste** = priorité la plus **forte** (principe LIFO).

---

## États d'une GPO

| État | Description |
|------|-------------|
| **Link Enabled** | Le lien GPO-OU est actif/inactif |
| **GPO Status Enabled** | La GPO entière est active/inactive |
| **Enforced** | Priorité absolue — ignore le Block Inheritance |
| **Block Inheritance** | Bloque l'héritage des GPO supérieures (sauf Enforced) |

---

## Filtrage de sécurité

> Condition d'application : l'objet AD doit avoir **2 droits** sur la GPO :  
> **Read** + **Apply Group Policy**

| Action | Méthode |
|--------|---------|
| GPO pour tout le monde | Laisser `Authenticated Users` (défaut) |
| Restreindre à un groupe | Retirer `Authenticated Users` → Ajouter groupe → Read + Apply |
| Exclure un groupe | Ajouter groupe → **Deny** Apply Group Policy |

> ⚠️ `Deny` prévaut toujours sur `Allow` — à utiliser avec précaution.

---

## GUI — GPMC (Group Policy Management Console)

### Ouvrir GPMC

```
Win + R → gpmc.msc
# ou
Gestionnaire de serveur → Outils → Gestion des stratégies de groupe
```

### Créer une GPO (GUI)

1. Dans GPMC, clic droit sur `Objets de stratégie de groupe` → `Nouveau`
2. Saisir le **nom** (ex: `GPO-Users-MapDrives`) → `OK`
3. La GPO est créée mais **non liée**

### Lier une GPO à une OU (GUI)

1. Clic droit sur l'**OU cible** → `Lier un objet de stratégie de groupe existant`
2. Sélectionner la GPO → `OK`

### Modifier une GPO (GUI)

1. Clic droit sur la GPO → `Modifier`
2. L'**Éditeur de gestion des stratégies de groupe** s'ouvre
3. Navigation dans l'arborescence :

```
GPO
+-- Configuration ordinateur
|   +-- Stratégies
|   +-- Préférences
+-- Configuration utilisateur
    +-- Stratégies
    +-- Préférences
        +-- Paramètres Windows
            +-- Mappages de lecteurs  ← Drive Maps (checkpoint 3)
```

### Procédure : GPO Drive-Mount (lecteurs réseau) — cas checkpoint

> Exercice type : créer une GPO qui monte les lecteurs E: et F: sur les clients.

1. Créer la GPO `GPO-Drive-Mount`
2. Clic droit → `Modifier`
3. Naviguer : `Configuration utilisateur` → `Préférences` → `Paramètres Windows` → `Mappages de lecteurs`
4. Clic droit → `Nouveau` → `Lecteur mappé`
5. Configurer :
   - **Action** : `Mettre à jour`
   - **Emplacement** : `\\serveur\DossiersCommuns`
   - **Lettre** : `E:`
   - **Label** : `DossiersCommuns`
   - Cocher `Afficher ce lecteur`
6. Répéter pour `F:` → `\\serveur\DossiersIndividuels\%LogonUser%`
7. Lier la GPO à l'OU des utilisateurs

### Forcer l'application / Vérifier

| Action | Commande / Outil |
|--------|-----------------|
| Forcer l'application GPO (client) | `gpupdate /force` |
| Voir les GPO appliquées (texte) | `gpresult /r` |
| Voir les GPO appliquées (HTML) | `gpresult /h C:\report.html` |
| Simuler les GPO (RSOP) | `rsop.msc` |
| Éditer stratégies locales | `gpedit.msc` |

---

## PowerShell — GPO

```powershell
# Lister toutes les GPO
Get-GPO -All

# Détails d'une GPO
Get-GPO -Name "GPO-Drive-Mount"

# Créer une nouvelle GPO
New-GPO -Name "GPO-Security-Password"

# Lier une GPO à une OU
New-GPLink -Name "GPO-Drive-Mount" `
           -Target "OU=LabUsers,DC=lab,DC=lan"

# Lier et activer immédiatement
New-GPLink -Name "GPO-Drive-Mount" `
           -Target "OU=LabUsers,DC=lab,DC=lan" `
           -LinkEnabled Yes

# Forcer l'application sur un poste
Invoke-GPUpdate -Computer "PC01" -Force

# Rapport HTML d'une GPO
Get-GPOReport -Name "GPO-Drive-Mount" -ReportType Html -Path "C:\gpo-report.html"

# Rapport de toutes les GPO
Get-GPOReport -All -ReportType Html -Path "C:\all-gpo.html"

# Résultats GPO appliquées (RSOP)
Get-GPResultantSetOfPolicy -ReportType Html -Path "C:\rsop.html"

# Supprimer une GPO
Remove-GPO -Name "GPO-Old"

# Supprimer un lien GPO-OU
Remove-GPLink -Name "GPO-Drive-Mount" -Target "OU=LabUsers,DC=lab,DC=lan"
```

---

## Bonnes pratiques

| Pratique | Statut |
|----------|--------|
| Ne jamais modifier `Default Domain Policy` | ❌ Interdit |
| Ne pas lier de GPO aux containers `CN=Users` / `CN=Computers` | ❌ Impossible |
| Nomenclature descriptive : `GPO-[Cible]-[Action]` | ✅ Obligatoire |
| Une GPO = une fonction (GPO modulaires) | ✅ Recommandé |
| Désactiver la section inutilisée (ordinateur ou utilisateur) | ✅ Performance |
| Supprimer le lien plutôt que désactiver | ✅ Clarté |
| Limiter le `Block Inheritance` | ⚠️ Parcimonie |
| Documenter chaque GPO | ✅ Obligatoire |

---

## ⚠️ À retenir absolument (checkpoints + REAC)

- `gpmc.msc` = console de gestion des GPO
- `gpedit.msc` = éditeur de stratégies **locales** uniquement
- `gpupdate /force` = forcer l'application sur le client après une modif
- `gpresult /r` = vérifier quelles GPO sont appliquées à un utilisateur/PC
- **LSDOU** : OU a toujours la priorité la plus forte
- **LIFO** sur une même OU : GPO du bas de liste = priorité forte
- **Enforced** contourne le `Block Inheritance`
- Filtrage sécurité : `Read` + `Apply Group Policy` tous les deux requis
- `Drive Maps` = `Configuration utilisateur` → `Préférences` → `Paramètres Windows` → `Mappages de lecteurs`
- Variable `%LogonUser%` = dossier individuel par utilisateur dans un chemin UNC
- GPO non applicables aux clients Linux → ignorées silencieusement
