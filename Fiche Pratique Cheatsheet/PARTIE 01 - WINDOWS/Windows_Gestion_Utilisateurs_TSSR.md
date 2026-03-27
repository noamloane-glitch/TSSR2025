# Windows — Gestion des utilisateurs (local)

> Gestion des comptes locaux via **PowerShell** (cmdlets `LocalUser` / `LocalGroup`).  
> Stockage des comptes : base **SAM** (`C:\Windows\System32\config\SAM`).

---

## Concepts clés

| Concept | Description |
|---------|-------------|
| **SAM** | Security Account Manager — base locale des comptes Windows |
| **SID** | Security Identifier — identifiant unique de chaque compte/groupe |
| **Administrateurs** | SID `S-1-5-32-544` — contrôle total |
| **Utilisateurs** | SID `S-1-5-32-545` — droits standards |
| **Invités** | SID `S-1-5-32-546` — accès minimal |

> ⚠️ La base SAM est **verrouillée** pendant que Windows tourne — inaccessible en direct.

---

## Commandes — Gestion des utilisateurs

| Commande | Rôle |
|----------|------|
| `Get-LocalUser` | Lister tous les comptes locaux |
| `Get-LocalUser -Name "user"` | Détails d'un compte |
| `New-LocalUser -Name "user" -Password (...)` | Créer un compte |
| `Set-LocalUser -Name "user" -Description "..."` | Modifier un compte |
| `Rename-LocalUser -Name "old" -NewName "new"` | Renommer un compte |
| `Enable-LocalUser -Name "user"` | Activer un compte |
| `Disable-LocalUser -Name "user"` | Désactiver un compte |
| `Remove-LocalUser -Name "user"` | Supprimer un compte |

---

## Commandes — Gestion des groupes

| Commande | Rôle |
|----------|------|
| `Get-LocalGroup` | Lister tous les groupes locaux |
| `New-LocalGroup -Name "grp"` | Créer un groupe |
| `Set-LocalGroup -Name "grp" -Description "..."` | Modifier un groupe |
| `Rename-LocalGroup -Name "old" -NewName "new"` | Renommer un groupe |
| `Remove-LocalGroup -Name "grp"` | Supprimer un groupe |
| `Get-LocalGroupMember -Group "grp"` | Lister les membres d'un groupe |
| `Add-LocalGroupMember -Group "grp" -Member "user"` | Ajouter un membre |
| `Remove-LocalGroupMember -Group "grp" -Member "user"` | Retirer un membre |

---

## Procédure : Créer un utilisateur complet

```powershell
# Étape 1 — Préparer le mot de passe (objet SecureString)
$mdp = ConvertTo-SecureString "P@ssw0rd123" -AsPlainText -Force

# Étape 2 — Créer le compte
New-LocalUser -Name "jdupont" `
              -Password $mdp `
              -FullName "Jean Dupont" `
              -Description "Technicien support"

# Étape 3 — Ajouter au groupe Utilisateurs (standard)
Add-LocalGroupMember -Group "Utilisateurs" -Member "jdupont"

# Étape 4 — (optionnel) Ajouter aux Administrateurs
Add-LocalGroupMember -Group "Administrateurs" -Member "jdupont"

# Étape 5 — Vérifier
Get-LocalUser -Name "jdupont" | Format-List *
Get-LocalGroupMember -Group "Utilisateurs"
```

---

## Procédure : Changer le mot de passe

```powershell
# Méthode 1 — Mot de passe en clair (scripts)
$mdp = ConvertTo-SecureString "NouveauMdp!" -AsPlainText -Force
Set-LocalUser -Name "jdupont" -Password $mdp

# Méthode 2 — Saisie interactive (plus sécurisée)
$mdp = Read-Host -AsSecureString "Nouveau mot de passe"
Set-LocalUser -Name "jdupont" -Password $mdp
```

---

## Procédure : Gérer l'expiration du mot de passe

```powershell
# Mot de passe n'expire jamais
Set-LocalUser -Name "jdupont" -PasswordNeverExpires $true

# Mot de passe expire (selon politique)
Set-LocalUser -Name "jdupont" -PasswordNeverExpires $false

# Forcer changement au prochain login (via net user)
net user jdupont /logonpasswordchg:yes
```

---

## Groupes importants à connaître

| Groupe | SID | Droits |
|--------|-----|--------|
| `Administrateurs` | `S-1-5-32-544` | Contrôle total |
| `Utilisateurs` | `S-1-5-32-545` | Droits standards |
| `Invités` | `S-1-5-32-546` | Accès minimal |
| `Utilisateurs du Bureau à distance` | `S-1-5-32-555` | Connexion RDP |
| `Opérateurs de sauvegarde` | `S-1-5-32-551` | Sauvegarde/restauration |
| `Opérateurs de configuration réseau` | `S-1-5-32-556` | Config réseau |

---

## Commandes utiles complémentaires

```powershell
# Lister les groupes d'un utilisateur
Get-LocalGroup | Where-Object {
    (Get-LocalGroupMember -Group $_.Name -ErrorAction SilentlyContinue).Name `
    -contains "PCLab\jdupont"
}

# Lister utilisateurs via WMI
Get-WmiObject Win32_UserAccount

# Lister groupes via WMI
Get-WmiObject Win32_Group

# Commandes net (alternatives CMD)
net user                          # Lister les utilisateurs
net user jdupont /add             # Créer un utilisateur
net user jdupont /delete          # Supprimer un utilisateur
net localgroup Administrateurs jdupont /add   # Ajouter à un groupe
```

---

## ⚠️ À retenir absolument

- `ConvertTo-SecureString` obligatoire pour passer un mot de passe à `New-LocalUser`
- `Add-LocalGroupMember` ne crée **pas** le compte — il doit déjà exister
- Compte désactivé ≠ compte supprimé → préférer `Disable-LocalUser` par sécurité
- La base **SAM** est verrouillée pendant l'exécution de Windows
- `net user` = alternative CMD/batch aux cmdlets PowerShell
- SID `S-1-5-32-544` = groupe **Administrateurs** — à connaître par cœur
