# AD-03 — Utilisateurs & Groupes

> Gestion centralisée des comptes et des accès dans Active Directory.  
> Principe fondamental : **jamais de permissions directes** sur un utilisateur → toujours via des groupes.

---

## Identifiants uniques d'un objet AD

| Identifiant | Portée | Immuable | Usage |
|-------------|--------|----------|-------|
| **GUID** | Forêt | ✅ Oui | Référence permanente (128 bits) |
| **SID** | Domaine | ❌ Non | Contrôle d'accès NTFS |
| **DN** (Distinguished Name) | Forêt | ❌ Non | Chemin LDAP complet |

> Exemple DN : `CN=Jean.Dupont,OU=Compta,OU=LabUsers,DC=lab,DC=lan`

---

## Utilisateurs — Attributs essentiels

| Attribut | Description |
|----------|-------------|
| `SamAccountName` | Identifiant de connexion Windows (max 20 car.) |
| `UserPrincipalName` | UPN — format `user@domaine.lan` |
| `DisplayName` | Nom affiché |
| `PasswordNeverExpires` | Mot de passe permanent |
| `Enabled` | Compte actif ou désactivé |
| `MemberOf` | Groupes d'appartenance |

---

## GUI — ADUC (Active Directory Users and Computers)

### Ouvrir ADUC

```
Gestionnaire de serveur → Outils → Utilisateurs et ordinateurs Active Directory
# ou
Win + R → dsa.msc
```

### Créer un utilisateur (GUI)

1. Naviguer jusqu'à l'**OU cible** dans l'arborescence
2. Clic droit sur l'OU → `Nouveau` → `Utilisateur`
3. Remplir : Prénom, Nom, **SamAccountName**, UPN
4. `Suivant` → Définir le **mot de passe** + options :

| Option | Usage |
|--------|-------|
| L'utilisateur doit changer le mot de passe... | Comptes standards |
| Le mot de passe n'expire jamais | Comptes de service |
| Le compte est désactivé | Pré-création |

5. `Suivant` → `Terminer`

### Modifier un utilisateur (GUI)

Double-clic sur l'utilisateur → onglets disponibles :

| Onglet | Contenu |
|--------|---------|
| Général | Nom, prénom, description, téléphone |
| Compte | Login, UPN, options mot de passe, expiration |
| Profil | Chemin profil itinérant, script de logon |
| Membre de | Groupes d'appartenance |
| Organisation | Titre, service, manager |

### Actions rapides (clic droit sur l'utilisateur)

| Action | Résultat |
|--------|----------|
| Désactiver le compte | Icône avec flèche ↓ rouge dans ADUC |
| Activer le compte | Réactiver un compte désactivé |
| Réinitialiser le mot de passe | Nouveau MDP + options |
| Déplacer | Changer d'OU |
| Ajouter à un groupe | Ajout rapide à un groupe |
| Supprimer | Suppression définitive |

> ⚠️ Bonne pratique départ employé : **Désactiver** → **Déplacer** dans DeactivatedUsers → **Supprimer** après délai

---

## GUI — Groupes

### Créer un groupe (GUI)

1. Clic droit sur l'OU cible → `Nouveau` → `Groupe`
2. Remplir :
   - **Nom du groupe** (ex: `Grp_Compta`)
   - **Étendue** : Domaine local / Globale / Universelle
   - **Type** : Sécurité / Distribution
3. `OK`

### Ajouter des membres à un groupe (GUI)

**Depuis le groupe :**
1. Double-clic sur le groupe → onglet `Membres` → `Ajouter`
2. Rechercher utilisateurs ou groupes → `OK`

**Depuis l'utilisateur :**
1. Double-clic sur l'utilisateur → onglet `Membre de` → `Ajouter`
2. Rechercher le groupe → `OK`

### Retirer un membre (GUI)

1. Double-clic sur le groupe → onglet `Membres`
2. Sélectionner l'utilisateur → `Supprimer` → `OK`

---

## PowerShell — Utilisateurs

```powershell
# Créer un utilisateur
New-ADUser -Name "Jean Dupont" `
           -SamAccountName "jean.dupont" `
           -UserPrincipalName "jean.dupont@lab.lan" `
           -Path "OU=Compta,OU=LabUsers,DC=lab,DC=lan" `
           -AccountPassword (ConvertTo-SecureString "P@ssw0rd!" -AsPlainText -Force) `
           -Enabled $true

# Lister tous les utilisateurs
Get-ADUser -Filter *

# Lister avec attributs spécifiques
Get-ADUser -Filter * -Properties DisplayName, MemberOf | Select Name, DisplayName

# Chercher un utilisateur
Get-ADUser -Identity "jean.dupont"
Get-ADUser -Filter {Name -like "Jean*"}

# Modifier un utilisateur
Set-ADUser -Identity "jean.dupont" -Title "Comptable" -Department "Compta"

# Activer / Désactiver un compte
Enable-ADAccount  -Identity "jean.dupont"
Disable-ADAccount -Identity "jean.dupont"

# Réinitialiser le mot de passe
Set-ADAccountPassword -Identity "jean.dupont" `
    -NewPassword (ConvertTo-SecureString "Nouveau@1" -AsPlainText -Force) -Reset

# Déplacer dans une autre OU
Move-ADObject -Identity "CN=Jean Dupont,OU=Compta,DC=lab,DC=lan" `
              -TargetPath "OU=DeactivatedUsers,DC=lab,DC=lan"

# Supprimer un utilisateur
Remove-ADUser -Identity "jean.dupont" -Confirm:$false
```

---

## PowerShell — Groupes

```powershell
# Créer un groupe
New-ADGroup -Name "Grp_Compta" `
            -GroupScope Global `
            -GroupCategory Security `
            -Path "OU=Groupes,DC=lab,DC=lan"

# Lister les groupes
Get-ADGroup -Filter * | Select Name, GroupScope, GroupCategory

# Ajouter un ou plusieurs membres
Add-ADGroupMember -Identity "Grp_Compta" -Members "jean.dupont","alice.martin"

# Ajouter un groupe dans un autre groupe (AGDLP)
Add-ADGroupMember -Identity "Grp_Compta_RW" -Members "Grp_Compta"

# Retirer un membre
Remove-ADGroupMember -Identity "Grp_Compta" -Members "jean.dupont" -Confirm:$false

# Voir les membres d'un groupe
Get-ADGroupMember -Identity "Grp_Compta"

# Voir les groupes d'un utilisateur
Get-ADUser -Identity "jean.dupont" -Properties MemberOf | Select -Expand MemberOf

# Supprimer un groupe
Remove-ADGroup -Identity "Grp_Compta" -Confirm:$false
```

---

## Types et étendues des groupes

### Types

| Type | Usage | SID attribué |
|------|-------|-------------|
| **Sécurité** (Security) | ✅ Permissions NTFS, GPO | ✅ Oui |
| **Distribution** | Email uniquement | ❌ Non |

### Étendues

| Étendue | Membres acceptés | Portée permissions | Rôle AGDLP |
|---------|-----------------|-------------------|------------|
| **Domain Local (DL)** | Tous domaines approuvés | Domaine local uniquement | Groupes de **droits** |
| **Global (G)** | Même domaine uniquement | Tous domaines approuvés | Groupes **métiers** |
| **Universal (U)** | Tous domaines forêt | Toute la forêt | Multi-domaines ⚠️ limiter |

---

## Méthode AGDLP

> **A**ccount → **G**lobal → **D**omain **L**ocal → **P**ermissions

### Règles fondamentales

| Règle | Statut |
|-------|--------|
| Utilisateurs membres de groupes **Global** (métiers) | ✅ |
| Groupes Global membres de groupes **Domain Local** (droits) | ✅ |
| Groupes Domain Local reçoivent les **permissions NTFS** | ✅ |
| Permissions directes sur un utilisateur | ❌ Jamais |
| Permissions directes sur un groupe Global | ❌ Jamais |

### Schéma AGDLP

```
Utilisateurs (A)      Groupe métier (G)      Groupe droits (DL)        Permissions (P)

Jean.Dupont  --+
               +--> Grp_Compta --+--> Grp_Compta_RW --> \\srv\Compta (RW)
Alice.Martin --+                 +--> Grp_Paye_R    --> \\srv\Paye   (R)
```

### Mise en oeuvre PowerShell (exemple complet AGDLP)

```powershell
# 1. Groupes metiers (Global)
New-ADGroup -Name "Grp_Compta" -GroupScope Global -GroupCategory Security `
            -Path "OU=Groupes,DC=lab,DC=lan"

# 2. Groupes de droits (Domain Local)
New-ADGroup -Name "Grp_Compta_RW" -GroupScope DomainLocal -GroupCategory Security `
            -Path "OU=Groupes,DC=lab,DC=lan"
New-ADGroup -Name "Grp_Paye_R" -GroupScope DomainLocal -GroupCategory Security `
            -Path "OU=Groupes,DC=lab,DC=lan"

# 3. Utilisateurs -> Groupe metier
Add-ADGroupMember -Identity "Grp_Compta" -Members "jean.dupont","alice.martin"

# 4. Groupe metier -> Groupes de droits
Add-ADGroupMember -Identity "Grp_Compta_RW" -Members "Grp_Compta"
Add-ADGroupMember -Identity "Grp_Paye_R"    -Members "Grp_Compta"

# 5. Permissions NTFS (via icacls)
icacls "\\srv\Compta" /grant "Grp_Compta_RW:(OI)(CI)M"
icacls "\\srv\Paye"   /grant "Grp_Paye_R:(OI)(CI)R"
```

### Nomenclature recommandée

| Type | Préfixe | Exemple |
|------|---------|---------|
| Groupe métier (Global) | `Grp_` | `Grp_Compta`, `Grp_RH` |
| Droits lecture (DL) | `Grp_<Res>_R` | `Grp_Compta_R` |
| Droits lecture/écriture (DL) | `Grp_<Res>_RW` | `Grp_Compta_RW` |
| Droits contrôle total (DL) | `Grp_<Res>_F` | `Grp_Compta_F` |

---

## ⚠️ À retenir absolument

- `dsa.msc` = raccourci pour ouvrir ADUC
- Compte désactivé = icône avec **flèche ↓** dans ADUC
- **AGDLP** = jamais de permissions directes sur un utilisateur
- Groupe **Global** = regroupement **métier** (qui sont les gens ?)
- Groupe **Domain Local** = regroupement **droits** (accès à quoi ?)
- Type **Distribution** = email uniquement, **pas de permissions NTFS possibles**
- `SamAccountName` = login Windows (max 20 caractères)
- `UserPrincipalName` = format `user@domaine.lan`
- Groupe **Universal** = répliqué Catalogue Global → à utiliser avec parcimonie
- Bonne pratique départ : **Désactiver** → **Déplacer** → **Supprimer**
