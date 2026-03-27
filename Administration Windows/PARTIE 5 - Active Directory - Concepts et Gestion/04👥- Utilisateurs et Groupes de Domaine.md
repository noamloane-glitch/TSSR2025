
> [!info] Vue d'ensemble
> Les utilisateurs et groupes constituent le cœur de la gestion des identités dans Active Directory. Cette section couvre la création, la configuration et l'organisation des comptes utilisateurs et groupes au niveau du domaine.

---

## 📑 Table des matières

- [[#🧑‍💼 Création d'Utilisateurs de Domaine|Création d'Utilisateurs de Domaine]]
- [[#🏷️ Attributs Utilisateurs|Attributs Utilisateurs]]
  - [[#UPN (User Principal Name)|UPN]]
  - [[#SamAccountName|SamAccountName]]
- [[#👥 Groupes de Sécurité et de Distribution|Groupes de Sécurité et de Distribution]]
- [[#🌐 Étendues de Groupe|Étendues de Groupe]]
- [[#📝 Stratégies de Nommage|Stratégies de Nommage]]
- [[#🔐 Groupes Intégrés du Domaine|Groupes Intégrés du Domaine]]

---

## 🧑‍💼 Création d'Utilisateurs de Domaine

### Méthodes de Création

Active Directory offre plusieurs méthodes pour créer des utilisateurs de domaine, chacune adaptée à des besoins spécifiques.

#### Via l'Interface Graphique (ADUC)

> [!example] Création via ADUC
> **Active Directory Users and Computers** est l'outil graphique principal pour la gestion quotidienne.
> 
> **Étapes :**
> 1. Ouvrir **ADUC** (`dsa.msc`)
> 2. Naviguer vers l'OU cible
> 3. Clic droit → **New** → **User**
> 4. Remplir les informations requises
> 5. Définir le mot de passe et les options

**Avantages :**
- Interface intuitive pour les débutants
- Visualisation de la structure organisationnelle
- Idéal pour créations ponctuelles

**Inconvénients :**
- Chronophage pour créations multiples
- Pas de traçabilité automatique
- Difficile à standardiser

#### Via PowerShell (Recommandé)

```powershell
# Création d'un utilisateur de domaine basique
New-ADUser -Name "Jean Dupont" `
           -GivenName "Jean" `
           -Surname "Dupont" `
           -SamAccountName "jdupont" `
           -UserPrincipalName "jdupont@entreprise.local" `
           -Path "OU=Utilisateurs,OU=Paris,DC=entreprise,DC=local" `
           -AccountPassword (ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force) `
           -Enabled $true `
           -ChangePasswordAtLogon $true

# Création avancée avec attributs complets
New-ADUser -Name "Marie Martin" `
           -GivenName "Marie" `
           -Surname "Martin" `
           -SamAccountName "mmartin" `
           -UserPrincipalName "mmartin@entreprise.local" `
           -DisplayName "MARTIN Marie" `
           -Description "Responsable RH" `
           -Title "Responsable Ressources Humaines" `
           -Department "RH" `
           -Office "Paris - Bureau 205" `
           -EmailAddress "marie.martin@entreprise.fr" `
           -MobilePhone "+33612345678" `
           -OfficePhone "+33140123456" `
           -Company "Entreprise SA" `
           -Manager "CN=Pierre Directeur,OU=Direction,DC=entreprise,DC=local" `
           -Path "OU=RH,OU=Services,DC=entreprise,DC=local" `
           -AccountPassword (ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force) `
           -Enabled $true `
           -ChangePasswordAtLogon $true
```

> [!tip] Bonnes Pratiques PowerShell
> - Utilisez le backtick (`) pour améliorer la lisibilité
> - Stockez les mots de passe dans des variables sécurisées
> - Créez des fonctions réutilisables pour standardiser
> - Journalisez toutes les créations dans des fichiers logs

#### Création en Masse via CSV

```powershell
# Exemple de fichier CSV (utilisateurs.csv)
# GivenName,Surname,SamAccountName,UPN,Department,Title,OU
# Jean,Dupont,jdupont,jdupont@entreprise.local,IT,Technicien,"OU=IT,DC=entreprise,DC=local"
# Marie,Martin,mmartin,mmartin@entreprise.local,RH,Manager,"OU=RH,DC=entreprise,DC=local"

# Script d'importation
Import-Csv -Path "C:\utilisateurs.csv" | ForEach-Object {
    $Password = ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force
    
    New-ADUser -Name "$($_.GivenName) $($_.Surname)" `
               -GivenName $_.GivenName `
               -Surname $_.Surname `
               -SamAccountName $_.SamAccountName `
               -UserPrincipalName $_.UPN `
               -Department $_.Department `
               -Title $_.Title `
               -Path $_.OU `
               -AccountPassword $Password `
               -Enabled $true `
               -ChangePasswordAtLogon $true
    
    Write-Host "Utilisateur $($_.SamAccountName) créé avec succès" -ForegroundColor Green
}
```

> [!warning] Pièges Courants
> - **Doublons** : Vérifiez toujours que le SamAccountName n'existe pas déjà
> - **Format CSV** : Respectez l'encodage UTF-8 pour les caractères spéciaux
> - **Chemins OU** : Validez l'existence des OU avant l'import
> - **Complexité mot de passe** : Respectez la stratégie de domaine

### Options de Compte Utilisateur

```powershell
# Désactiver un compte
Disable-ADAccount -Identity "jdupont"

# Activer un compte
Enable-ADAccount -Identity "jdupont"

# Définir l'expiration du compte
Set-ADUser -Identity "jdupont" -AccountExpirationDate "31/12/2024"

# Configurer les horaires de connexion (exemple : Lundi-Vendredi 8h-18h)
$logonHours = [byte[]](0) * 21
# Configuration des heures (code binaire représentant les plages horaires)
Set-ADUser -Identity "jdupont" -Replace @{logonHours=$logonHours}

# Limiter les stations de travail autorisées
Set-ADUser -Identity "jdupont" -LogonWorkstations "PC-001,PC-002,PC-003"

# Configurer "L'utilisateur ne peut pas changer son mot de passe"
Set-ADUser -Identity "jdupont" -CannotChangePassword $true

# Configurer "Le mot de passe n'expire jamais"
Set-ADUser -Identity "jdupont" -PasswordNeverExpires $true

# Configurer "Le compte est sensible et ne peut pas être délégué"
Set-ADUser -Identity "jdupont" -AccountNotDelegated $true
```

> [!info] Pourquoi ces Options ?
> - **Expiration de compte** : Idéal pour comptes temporaires (stagiaires, contractuels)
> - **Horaires de connexion** : Sécurise les comptes en limitant les accès aux heures de travail
> - **Stations autorisées** : Empêche l'utilisation de comptes depuis des postes non autorisés
> - **Mot de passe n'expire jamais** : Utilisé pour comptes de service (à éviter pour utilisateurs normaux)

---

## 🏷️ Attributs Utilisateurs

Les attributs définissent les propriétés et caractéristiques d'un compte utilisateur. Deux attributs sont particulièrement critiques pour l'authentification.

### UPN (User Principal Name)

Le **UPN** est l'identifiant moderne de connexion, similaire à une adresse email.

> [!info] Structure du UPN
> **Format** : `utilisateur@suffixe-upn`
> 
> **Exemples :**
> - `jdupont@entreprise.local`
> - `jean.dupont@entreprise.fr`
> - `j.dupont@filiale.entreprise.com`

#### Composants du UPN

| Composant | Description | Exemple |
|-----------|-------------|---------|
| **Préfixe** | Identifiant unique de l'utilisateur | `jdupont` |
| **@ (séparateur)** | Délimiteur obligatoire | `@` |
| **Suffixe** | Domaine ou suffixe UPN alternatif | `entreprise.local` |

#### Suffixes UPN

```powershell
# Lister les suffixes UPN disponibles dans la forêt
Get-ADForest | Select-Object -ExpandProperty UPNSuffixes

# Ajouter un suffixe UPN alternatif
Set-ADForest -UPNSuffixes @{Add="entreprise.fr","filiale.entreprise.com"}

# Modifier le UPN d'un utilisateur
Set-ADUser -Identity "jdupont" -UserPrincipalName "jean.dupont@entreprise.fr"

# Modifier en masse les UPN d'une OU
Get-ADUser -Filter * -SearchBase "OU=IT,DC=entreprise,DC=local" | ForEach-Object {
    $newUPN = $_.SamAccountName + "@entreprise.fr"
    Set-ADUser $_ -UserPrincipalName $newUPN
    Write-Host "UPN modifié : $($_.Name) -> $newUPN"
}
```

> [!tip] Pourquoi Utiliser des Suffixes UPN Alternatifs ?
> - **Expérience utilisateur** : `jdupont@entreprise.fr` est plus convivial que `jdupont@entreprise.local`
> - **Cohérence email** : Aligne l'UPN avec l'adresse email réelle
> - **Multi-domaines** : Permet de distinguer les utilisateurs de différentes entités
> - **Office 365/Azure AD** : Facilite la synchronisation avec des UPN publics

#### Contraintes et Règles UPN

> [!warning] Limitations UPN
> - **Unicité** : Le UPN doit être unique dans toute la forêt AD
> - **Longueur** : Maximum 1024 caractères (recommandé < 64)
> - **Caractères interdits** : `[ ] : ; | = + * ? < > , /`
> - **Sensibilité à la casse** : Non sensible, mais conserve la casse saisie

```powershell
# Vérifier l'unicité d'un UPN avant création
$upn = "jdupont@entreprise.local"
if (Get-ADUser -Filter "UserPrincipalName -eq '$upn'") {
    Write-Host "ERREUR : Ce UPN existe déjà !" -ForegroundColor Red
} else {
    Write-Host "UPN disponible" -ForegroundColor Green
}
```

### SamAccountName

Le **SamAccountName** (Security Account Manager Account Name) est l'identifiant historique hérité de Windows NT, toujours obligatoire dans AD.

> [!info] Caractéristiques du SamAccountName
> **Rôle** : Identifiant de connexion pré-Windows 2000
> 
> **Format** : `DOMAINE\SamAccountName`
> 
> **Exemple** : `ENTREPRISE\jdupont`

#### Contraintes SamAccountName

| Contrainte | Détail |
|------------|--------|
| **Longueur maximale** | 20 caractères maximum |
| **Unicité** | Unique dans le domaine uniquement |
| **Sensibilité casse** | Non sensible à la casse |
| **Caractères interdits** | `" / \ [ ] : ; | = , + * ? < > @` |
| **Caractères recommandés** | Lettres, chiffres, tirets, underscores |

```powershell
# Création avec SamAccountName explicite
New-ADUser -Name "Jean Dupont" `
           -SamAccountName "jdupont" `
           -UserPrincipalName "jdupont@entreprise.local"

# Renommer un SamAccountName (ATTENTION : impact majeur)
Set-ADUser -Identity "jdupont" -SamAccountName "jean.dupont"

# Vérifier la disponibilité d'un SamAccountName
$sam = "jdupont"
if (Get-ADUser -Filter "SamAccountName -eq '$sam'") {
    Write-Host "ERREUR : Ce SamAccountName existe déjà !" -ForegroundColor Red
} else {
    Write-Host "SamAccountName disponible" -ForegroundColor Green
}

# Générer automatiquement un SamAccountName unique
function New-UniqueSamAccountName {
    param([string]$FirstName, [string]$LastName)
    
    $base = ($FirstName.Substring(0,1) + $LastName).ToLower()
    $sam = $base
    $counter = 1
    
    while (Get-ADUser -Filter "SamAccountName -eq '$sam'" -ErrorAction SilentlyContinue) {
        $sam = $base + $counter
        $counter++
    }
    
    # Tronquer à 20 caractères si nécessaire
    if ($sam.Length -gt 20) {
        $sam = $sam.Substring(0, 20)
    }
    
    return $sam
}

# Utilisation
$uniqueSam = New-UniqueSamAccountName -FirstName "Jean" -LastName "Dupont"
```

> [!warning] Pièges Courants avec SamAccountName
> - **Limite de 20 caractères** : Troncature automatique possible, créant des doublons
> - **Modification hasardeuse** : Change l'identifiant de connexion, impacte les scripts et applications
> - **Caractères spéciaux** : Les accents et caractères spéciaux posent problème
> - **Confusion avec UPN** : Ce sont deux attributs distincts avec des règles différentes

#### Comparaison UPN vs SamAccountName

| Aspect | UPN | SamAccountName |
|--------|-----|----------------|
| **Longueur max** | 1024 caractères | 20 caractères |
| **Portée unicité** | Forêt entière | Domaine uniquement |
| **Format moderne** | ✅ Oui | ❌ Non (hérité) |
| **Authentification Kerberos** | ✅ Oui | ✅ Oui |
| **Authentification NTLM** | ❌ Non | ✅ Oui |
| **Office 365 / Azure AD** | ✅ Requis | ⚠️ Synchronisé mais secondaire |
| **Convivialité** | ✅ Haute (format email) | ⚠️ Moyenne |

> [!tip] Recommandations
> - **Standardisez** les deux attributs selon une convention cohérente
> - **Privilégiez l'UPN** pour les communications utilisateurs
> - **Gardez le SamAccountName court** et sans caractères spéciaux
> - **Documentez** votre convention de nommage dans vos procédures

---

## 👥 Groupes de Sécurité et de Distribution

Les groupes permettent d'organiser les utilisateurs et de gérer collectivement les permissions et la distribution d'emails.

### Types de Groupes

Active Directory propose deux types principaux de groupes, chacun ayant un rôle distinct.

#### Groupes de Sécurité (Security Groups)

> [!info] Groupes de Sécurité
> **Objectif principal** : Gestion des permissions et droits d'accès
> 
> **Fonctionnalités** :
> - Attribution de permissions sur ressources (fichiers, dossiers, imprimantes)
> - Assignation de droits utilisateurs
> - Filtrage de stratégies de groupe (GPO)
> - Peuvent également servir de listes de distribution

```powershell
# Créer un groupe de sécurité
New-ADGroup -Name "GRP-SEC-Compta-RW" `
            -GroupScope Global `
            -GroupCategory Security `
            -Path "OU=Groupes,DC=entreprise,DC=local" `
            -Description "Accès en lecture/écriture au partage Comptabilité"

# Créer un groupe pour GPO filtering
New-ADGroup -Name "GPO-Bloquer-USB" `
            -GroupScope Global `
            -GroupCategory Security `
            -Path "OU=Groupes-GPO,DC=entreprise,DC=local" `
            -Description "Utilisateurs concernés par le blocage USB"
```

**Utilisations typiques :**
- `GRP-SEC-Finance-R` : Lecture seule sur dossier Finance
- `GRP-SEC-Finance-RW` : Lecture/écriture sur dossier Finance
- `GPO-Fonds-Ecran-Marketing` : Application GPO fond d'écran Marketing
- `GRP-Admins-Serveurs` : Droits administrateurs sur serveurs

#### Groupes de Distribution (Distribution Groups)

> [!info] Groupes de Distribution
> **Objectif principal** : Distribution d'emails uniquement
> 
> **Fonctionnalités** :
> - Listes de diffusion pour messagerie
> - Aucun droit de sécurité
> - Utilisés principalement avec Exchange Server

```powershell
# Créer un groupe de distribution
New-ADGroup -Name "DL-Tous-Commercial" `
            -GroupScope Universal `
            -GroupCategory Distribution `
            -Path "OU=Groupes-Distribution,DC=entreprise,DC=local" `
            -Description "Liste de diffusion - Équipe commerciale"

# Créer une liste de distribution départementale
New-ADGroup -Name "DL-Direction-Generale" `
            -GroupScope Universal `
            -GroupCategory Distribution `
            -Path "OU=Groupes-Distribution,DC=entreprise,DC=local" `
            -Description "Liste de diffusion - Direction"
```

**Utilisations typiques :**
- `DL-Tous-Employes` : Tous les employés
- `DL-Dept-RH` : Département RH
- `DL-Equipe-Projet-Alpha` : Équipe projet spécifique

> [!tip] Quand Utiliser Quel Type ?
> - **Groupes de sécurité** : Par défaut, car ils peuvent servir aux deux usages
> - **Groupes de distribution** : Uniquement si vous voulez explicitement empêcher leur utilisation pour les permissions
> - **Bonne pratique** : Créer des groupes de sécurité et les activer pour la messagerie si nécessaire

### Gestion des Membres

```powershell
# Ajouter un membre à un groupe
Add-ADGroupMember -Identity "GRP-SEC-Compta-RW" -Members "jdupont"

# Ajouter plusieurs membres
Add-ADGroupMember -Identity "GRP-SEC-Compta-RW" -Members "jdupont","mmartin","pdubois"

# Ajouter un groupe à un autre groupe (imbrication)
Add-ADGroupMember -Identity "GRP-SEC-Finance-Tous" -Members "GRP-SEC-Compta-RW"

# Retirer un membre
Remove-ADGroupMember -Identity "GRP-SEC-Compta-RW" -Members "jdupont" -Confirm:$false

# Lister les membres d'un groupe
Get-ADGroupMember -Identity "GRP-SEC-Compta-RW"

# Lister les membres récursivement (avec membres des groupes imbriqués)
Get-ADGroupMember -Identity "GRP-SEC-Finance-Tous" -Recursive

# Lister les groupes dont un utilisateur est membre
Get-ADPrincipalGroupMembership -Identity "jdupont" | Select-Object Name

# Vider complètement un groupe
Get-ADGroupMember -Identity "GRP-Ancien-Projet" | ForEach-Object {
    Remove-ADGroupMember -Identity "GRP-Ancien-Projet" -Members $_ -Confirm:$false
}
```

> [!warning] Attention à l'Imbrication
> - Les groupes imbriqués peuvent compliquer l'audit et le dépannage
> - Documentez toujours vos stratégies d'imbrication
> - Limitez la profondeur d'imbrication (recommandé : max 3 niveaux)

### Conversion de Type

```powershell
# Convertir un groupe de distribution en groupe de sécurité
Set-ADGroup -Identity "DL-Tous-Commercial" -GroupCategory Security

# Convertir un groupe de sécurité en groupe de distribution
Set-ADGroup -Identity "GRP-SEC-Ancien" -GroupCategory Distribution
```

> [!info] Pourquoi Convertir ?
> - **Distribution → Sécurité** : Le groupe doit maintenant gérer des permissions
> - **Sécurité → Distribution** : Le groupe ne sert plus qu'à la messagerie (rare)
> 
> ⚠️ La conversion peut impacter les permissions existantes !

---

## 🌐 Étendues de Groupe

L'**étendue** (scope) d'un groupe détermine sa portée d'utilisation dans la structure Active Directory (domaine unique vs multi-domaines).

### Types d'Étendues

#### 1. Groupe Local de Domaine (Domain Local)

> [!info] Domain Local Group
> **Portée** : Utilisable uniquement dans le domaine où il est créé
> 
> **Peut contenir** :
> - Utilisateurs de n'importe quel domaine de la forêt
> - Groupes Globaux de n'importe quel domaine
> - Groupes Universels de n'importe quel domaine
> - Autres groupes Domain Local du même domaine
> 
> **Peut être utilisé pour** :
> - Permissions sur ressources du domaine local uniquement

```powershell
# Créer un groupe Domain Local
New-ADGroup -Name "DL-Partage-Compta-RW" `
            -GroupScope DomainLocal `
            -GroupCategory Security `
            -Path "OU=Groupes,DC=paris,DC=entreprise,DC=local" `
            -Description "Accès RW au partage Compta (Domaine Paris)"
```

**Scénario d'utilisation typique :**
```
Ressource (Serveur de fichiers Paris)
  ↓ Permission accordée à
[DL-Partage-Finance-RW] (Domain Local - Domaine Paris)
  ↓ Contient
[GG-Finance-Paris] (Global - Domaine Paris)
[GG-Finance-Lyon] (Global - Domaine Lyon)
  ↓ Contiennent
Utilisateurs de chaque domaine
```

#### 2. Groupe Global (Global)

> [!info] Global Group
> **Portée** : Peut être utilisé dans tout domaine de la forêt
> 
> **Peut contenir** :
> - Utilisateurs du même domaine uniquement
> - Autres groupes Globaux du même domaine uniquement
> 
> **Peut être utilisé pour** :
> - Permissions dans n'importe quel domaine de la forêt
> - Membre de groupes Domain Local ou Universal

```powershell
# Créer un groupe Global
New-ADGroup -Name "GG-Compta-Paris" `
            -GroupScope Global `
            -GroupCategory Security `
            -Path "OU=Groupes,DC=paris,DC=entreprise,DC=local" `
            -Description "Équipe Comptabilité de Paris"
```

**Caractéristique clé :** Représente un ensemble d'utilisateurs d'un domaine qui peut être utilisé partout dans la forêt.

#### 3. Groupe Universel (Universal)

> [!info] Universal Group
> **Portée** : Utilisable dans toute la forêt
> 
> **Peut contenir** :
> - Utilisateurs de n'importe quel domaine de la forêt
> - Groupes Globaux de n'importe quel domaine
> - Autres groupes Universels
> 
> **Peut être utilisé pour** :
> - Permissions dans n'importe quel domaine de la forêt

```powershell
# Créer un groupe Universal
New-ADGroup -Name "UG-Direction-Generale" `
            -GroupScope Universal `
            -GroupCategory Security `
            -Path "OU=Groupes,DC=entreprise,DC=local" `
            -Description "Direction générale - Tous sites"
```

> [!warning] Attention au Catalogue Global
> - Les groupes Universal sont répliqués dans le **Catalogue Global**
> - Chaque modification de membre génère une réplication complète du groupe
> - Utilisez avec parcimonie pour éviter un trafic réseau excessif
> - Ne pas utiliser pour des groupes à membres très changeants

### Tableau Comparatif des Étendues

| Caractéristique | Domain Local | Global | Universal |
|----------------|--------------|---------|-----------|
| **Membres possibles** | Tout domaine | Même domaine uniquement | Tout domaine |
| **Utilisable où ?** | Domaine local | Toute la forêt | Toute la forêt |
| **Réplication** | Domaine uniquement | Domaine uniquement | Catalogue Global |
| **Usage typique** | Permissions ressources | Groupes utilisateurs | Groupes inter-domaines |
| **Imbrication** | Peut contenir G, U, DL | Peut contenir G du même domaine | Peut contenir G, U |

### Stratégie AGDLP / AGUDLP

> [!tip] Méthode AGDLP (Best Practice Microsoft)
> **A**ccounts → **G**lobal groups → **D**omain **L**ocal groups → **P**ermissions
> 
> **Comment ça fonctionne :**
> 1. **Ajouter** les comptes utilisateurs (**A**) dans des groupes **G**lobaux
> 2. **Ajouter** les groupes **G**lobaux dans des groupes **D**omain **L**ocal
> 3. **Attribuer** les **P**ermissions aux groupes **D**omain **L**ocal

**Exemple concret :**

```powershell
# 1. Créer un groupe Global pour les utilisateurs
New-ADGroup -Name "GG-Comptables-Paris" `
            -GroupScope Global `
            -GroupCategory Security `
            -Path "OU=Groupes,DC=entreprise,DC=local"

# 2. Ajouter les utilisateurs au groupe Global
Add-ADGroupMember -Identity "GG-Comptables-Paris" -Members "jdupont","mmartin"

# 3. Créer un groupe Domain Local pour la ressource
New-ADGroup -Name "DL-Partage-Compta-RW" `
            -GroupScope DomainLocal `
            -GroupCategory Security `
            -Path "OU=Groupes,DC=entreprise,DC=local"

# 4. Ajouter le groupe Global au groupe Domain Local
Add-ADGroupMember -Identity "DL-Partage-Compta-RW" -Members "GG-Comptables-Paris"

# 5. Attribuer les permissions sur la ressource au groupe Domain Local
# (Ceci se fait généralement via l'interface ou icacls)
```

> [!tip] Variante AGUDLP (avec Universal)
> **A**ccounts → **G**lobal → **U**niversal → **D**omain **L**ocal → **P**ermissions
> 
> Utilisé dans les environnements multi-domaines complexes :
> - Groupes **G**lobaux par domaine
> - Groupes **U**niversels pour regrouper plusieurs domaines
> - Groupes **D**omain **L**ocal pour les permissions

**Avantages de cette stratégie :**
- ✅ Flexibilité dans les environnements multi-domaines
- ✅ Facilite la gestion des permissions
- ✅ Simplifie l'audit et le dépannage
- ✅ Optimise la réplication

### Modification d'Étendue

```powershell
# Convertir Domain Local en Universal (si le groupe ne contient pas d'autres DL)
Set-ADGroup -Identity "DL-Ancien" -GroupScope Universal

# Convertir Global en Universal
Set-ADGroup -Identity "GG-Ancien" -GroupScope Universal

# Convertir Universal en Domain Local
Set-ADGroup -Identity "UG-Ancien" -GroupScope DomainLocal

# Convertir Universal en Global (si membres uniquement du même domaine)
Set-ADGroup -Identity "UG-Ancien" -GroupScope Global
```

> [!warning] Restrictions de Conversion
> - **DL → Global** : ❌ Impossible directement (passer par Universal)
> - **Global → DL** : ❌ Impossible directement (passer par Universal)
> - **DL → Universal** : ✅ Si ne contient pas d'autres groupes DL
> - **Global → Universal** : ✅ Toujours possible
> - **Universal → DL** : ✅ Toujours possible
> - **Universal → Global** : ✅ Si tous les membres sont du même domaine

---

## 📝 Stratégies de Nommage

Une convention de nommage cohérente est essentielle pour maintenir un annuaire Active Directory organisé et compréhensible.

### Principes Généraux

> [!tip] Règles d'Or du Nommage
> 1. **Cohérence** : Appliquez la même logique partout
> 2. **Clarté** : Le nom doit indiquer immédiatement le rôle/objectif
> 3. **Évolutivité** : Prévoyez la croissance de l'organisation
> 4. **Documentation** : Documentez votre convention dans un référentiel accessible

### Convention pour Utilisateurs

#### Format SamAccountName

```
Format recommandé : [prénom].[nom]
Exemples : 
- jean.dupont
- marie.martin
- pierre.delarue

Alternative courte : [initiale][nom]
Exemples :
- jdupont
- mmartin
- pdelarue

Gestion des doublons :
- jean.dupont2
- j.dupont2
- jdupont01
```

```powershell
# Fonction de génération de SamAccountName avec gestion doublons
function New-StandardSamAccountName {
    param(
        [string]$FirstName,
        [string]$LastName
    )
    
    # Normalisation (suppression accents, minuscules)
    $firstName = $FirstName.ToLower() -replace '[éèêë]','e' -replace '[àâä]','a' -replace '[îï]','i' -replace '[ôö]','o' -replace '[ùûü]','u'
    $lastName = $LastName.ToLower() -replace '[éèêë]','e' -replace '[àâä]','a' -replace '[îï]','i' -replace '[ôö]','o' -replace '[ùûü]','u'
    
    # Génération base
    $base = "$firstName.$lastName"
    $sam = $base
    $counter = 2
    
    # Gestion doublons
    while (Get-ADUser -Filter "SamAccountName -eq '$sam'" -ErrorAction SilentlyContinue) {
        $sam = "$base$counter"
        $counter++
    }
    
    # Tronquer si nécessaire (max 20 caractères)
    if ($sam.Length -gt 20) {
        $sam = $sam.Substring(0, 20)
    }
    
    return $sam
}
```

#### Format UPN

```
Format recommandé : [SamAccountName]@[domaine-public.com]
Exemples :
- jean.dupont@entreprise.fr
- marie.martin@entreprise.com
- pierre.delarue@filiale.entreprise.fr

Avantages :
- Cohérence avec SamAccountName
- Alignement possible avec adresse email
- Facilité de mémorisation
```

#### Format DisplayName

```
Format recommandé : [NOM] [Prénom]
Exemples :
- DUPONT Jean
- MARTIN Marie
- DE LA RUE Pierre

Alternative : [Prénom] [NOM]
Exemples :
- Jean DUPONT
- Marie MARTIN
- Pierre DE LA RUE

Raison : 
- Tri alphabétique efficace
- Lisibilité dans les listes
- Convention professionnelle française
```

### Convention pour Groupes

#### Préfixes par Type et Étendue

```
Format : [TYPE]-[ÉTENDUE]-[Description]-[Permission]

TYPE :
- GRP : Groupe de sécurité générique
- DL  : Distribution List (groupe de distribution)
- GPO : Groupe pour filtrage GPO
- APP : Groupe applicatif

ÉTENDUE :
- DL : Domain Local
- G  : Global  
- U  : Universal

Exemples :
- GRP-G-Finance-Paris
- GRP-DL-Partage-Compta-RW
- DL-U-Direction-Generale
- GPO-G-Bloquer-USB
- APP-DL-SAP-Users
```

```powershell
# Exemples de création avec convention stricte
New-ADGroup -Name "GRP-G-IT-Paris" `
            -GroupScope Global `
            -GroupCategory Security `
            -Description "Équipe IT du site de Paris"

New-ADGroup -Name "GRP-DL-FS01-Projets-RW" `
            -GroupScope DomainLocal `
            -GroupCategory Security `
            -Description "Accès R/W au partage Projets sur FS01"

New-ADGroup -Name "DL-U-Marketing-France" `
            -GroupScope Universal `
            -GroupCategory Distribution `
            -Description "Liste diffusion Marketing France"

New-ADGroup -Name "GPO-G-Wallpaper-Direction" `
            -GroupScope Global `
            -GroupCategory Security `
            -Description "Application GPO fond écran Direction"
```

#### Suffixes de Permission (pour groupes ressources)

| Suffixe | Signification | Usage |
|---------|---------------|-------|
| **-R** | Read (Lecture) | Lecture seule |
| **-RW** | Read/Write (Lecture/Écriture) | Modification autorisée |
| **-M** | Modify (Modification) | Modification + suppression |
| **-FC** | Full Control (Contrôle total) | Tous les droits |
| **-List** | List (Liste) | Liste uniquement (sans lecture) |
| **-Contrib** | Contribute (Contribution) | Ajout de fichiers uniquement |

```
Exemples :
- GRP-DL-Partage-Finance-R
- GRP-DL-Partage-Finance-RW
- GRP-DL-Partage-Finance-FC
- GRP-DL-Imprimante-RH-Print
```

### Convention pour Comptes de Service

```
Format : SVC-[Application]-[Fonction]

Exemples :
- SVC-SQL-Engine
- SVC-IIS-AppPool
- SVC-Backup-Agent
- SVC-Exchange-Mailbox
- SVC-SCCM-Network

Caractéristiques :
- Préfixe SVC pour identification rapide
- Nom d'application explicite
- Fonction du service
- Ne JAMAIS utiliser pour connexion humaine
```

```powershell
# Créer un compte de service
New-ADUser -Name "SVC-SQL-Engine" `
           -SamAccountName "SVC-SQL-Engine" `
           -UserPrincipalName "SVC-SQL-Engine@entreprise.local" `
           -Description "Compte de service SQL Server Database Engine" `
           -Path "OU=Service-Accounts,DC=entreprise,DC=local" `
           -AccountPassword (ConvertTo-SecureString "P@ssw0rdComplexe!" -AsPlainText -Force) `
           -Enabled $true `
           -PasswordNeverExpires $true `
           -CannotChangePassword $true
```

> [!warning] Sécurité des Comptes de Service
> - Utilisez des mots de passe très complexes (25+ caractères)
> - Documentez chaque compte (application, serveur, contact)
> - Auditez régulièrement leur utilisation
> - Envisagez les **gMSA** (Group Managed Service Accounts) pour une sécurité renforcée

### Convention pour Comptes Administrateurs

```
Format : [prefix]-[utilisateur]

Préfixes :
- ADM-  : Administrateur domaine/serveur
- LADM- : Administrateur local
- DA-   : Domain Admin (déconseillé d'utiliser réellement)

Exemples :
- ADM-jdupont (compte admin de Jean Dupont)
- LADM-technicien (admin local pour technicien)
- ADM-srvmgmt (admin serveurs)

Règle : 
- JAMAIS utiliser le compte admin pour usage quotidien
- Comptes admin séparés des comptes utilisateurs standard
```

```powershell
# Créer un compte administrateur
New-ADUser -Name "ADM-Jean Dupont" `
           -GivenName "Jean" `
           -Surname "Dupont (Admin)" `
           -SamAccountName "ADM-jdupont" `
           -UserPrincipalName "ADM-jdupont@entreprise.local" `
           -Description "Compte administrateur de Jean Dupont" `
           -Path "OU=Admin-Accounts,DC=entreprise,DC=local" `
           -AccountPassword (ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force) `
           -Enabled $true `
           -ChangePasswordAtLogon $true

# Ajouter aux groupes d'administration appropriés
Add-ADGroupMember -Identity "Server Operators" -Members "ADM-jdupont"
```

### Documentation de la Convention

> [!tip] Créer un Document de Référence
> Créez et maintenez un document centralisé contenant :
> 
> **Pour les utilisateurs :**
> - Format SamAccountName
> - Format UPN et suffixes autorisés
> - Format DisplayName
> - Gestion des doublons
> - Cas particuliers (noms composés, caractères spéciaux)
> 
> **Pour les groupes :**
> - Signification des préfixes
> - Convention d'étendue
> - Suffixes de permission
> - Exemples par catégorie
> 
> **Pour les comptes spéciaux :**
> - Comptes de service
> - Comptes administrateurs
> - Comptes de test
> 
> **Processus :**
> - Qui valide les exceptions ?
> - Comment demander un nom spécifique ?
> - Procédure de renommage

---

## 🔐 Groupes Intégrés du Domaine

Active Directory crée automatiquement des groupes avec des permissions prédéfinies. Ces groupes intégrés simplifient l'administration mais doivent être utilisés avec précaution.

### Groupes d'Administration

#### Administrators (Administrateurs)

> [!warning] Groupe le Plus Puissant
> **Portée** : Contrôle total sur tout le domaine
> 
> **Capacités** :
> - Administration complète de tous les contrôleurs de domaine
> - Modification de tous les objets du domaine
> - Prise de possession de tout objet
> - Installation de services et pilotes
> 
> **Membres par défaut** :
> - Administrator (compte intégré)
> - Domain Admins
> - Enterprise Admins (seulement sur le DC racine de forêt)

```powershell
# Lister les membres du groupe Administrators
Get-ADGroupMember -Identity "Administrators"

# Ajouter un utilisateur (ATTENTION : Très sensible !)
Add-ADGroupMember -Identity "Administrators" -Members "ADM-jdupont"

# Retirer un utilisateur
Remove-ADGroupMember -Identity "Administrators" -Members "ADM-jdupont" -Confirm:$false
```

> [!warning] Bonnes Pratiques
> - ❌ **NE JAMAIS** ajouter des comptes utilisateurs normaux
> - ✅ Utiliser uniquement pour des comptes administrateurs dédiés
> - ✅ Auditer régulièrement les membres
> - ✅ Utiliser des comptes avec principe de moindre privilège

#### Domain Admins

> [!info] Domain Admins
> **Portée** : Administration complète du domaine
> 
> **Capacités** :
> - Membre du groupe Administrators sur tous les ordinateurs du domaine
> - Administration de tous les contrôleurs de domaine
> - Modification de la stratégie de domaine
> 
> **Membre par défaut** :
> - Administrator (compte intégré du domaine)

```powershell
# Lister les Domain Admins
Get-ADGroupMember -Identity "Domain Admins"

# Vérifier si un utilisateur est Domain Admin
$user = "ADM-jdupont"
$isDomainAdmin = (Get-ADUser $user -Properties MemberOf).MemberOf -match "CN=Domain Admins"
if ($isDomainAdmin) {
    Write-Host "$user est Domain Admin" -ForegroundColor Red
}
```

> [!tip] Utilisation Recommandée
> - Réservé aux administrateurs de domaine uniquement
> - Limiter le nombre de membres (5-10 maximum)
> - Utiliser uniquement quand nécessaire (principe de moindre privilège)
> - Envisager des groupes avec moins de privilèges pour certaines tâches

#### Enterprise Admins

> [!warning] Groupe de Forêt - Extrêmement Puissant
> **Portée** : Administration de toute la forêt AD
> 
> **Capacités** :
> - Administration de tous les domaines de la forêt
> - Modification du schéma Active Directory
> - Ajout/suppression de domaines
> - Configuration de la forêt
> 
> **Existe** : Uniquement dans le domaine racine de la forêt
> 
> **Membre par défaut** :
> - Administrator (du domaine racine uniquement)

```powershell
# Lister Enterprise Admins (depuis le domaine racine)
Get-ADGroupMember -Identity "Enterprise Admins" -Server "dc-root.entreprise.local"
```

> [!warning] Sécurité Critique
> - Utilisé uniquement pour modifications structurelles majeures
> - Doit rester vide 99% du temps
> - Ajouter temporairement quand nécessaire, puis retirer immédiatement
> - Auditer chaque modification avec journalisation

#### Schema Admins

> [!info] Schema Admins
> **Portée** : Modification du schéma Active Directory
> 
> **Capacités** :
> - Seul groupe autorisé à modifier le schéma AD
> - Ajout de nouveaux attributs et classes d'objets
> - Extension du schéma pour applications tierces
> 
> **Membre par défaut** :
> - Administrator (du domaine racine)

```powershell
# Lister Schema Admins
Get-ADGroupMember -Identity "Schema Admins" -Server "dc-root.entreprise.local"

# Ajouter temporairement pour une modification schéma
Add-ADGroupMember -Identity "Schema Admins" -Members "ADM-schema" -Server "dc-root.entreprise.local"

# RETIRER immédiatement après l'opération
Remove-ADGroupMember -Identity "Schema Admins" -Members "ADM-schema" -Confirm:$false -Server "dc-root.entreprise.local"
```

### Groupes d'Opérateurs

#### Server Operators

> [!info] Server Operators
> **Portée** : Administration des serveurs sans droits complets
> 
> **Capacités** :
> - Connexion interactive sur contrôleurs de domaine
> - Gestion des partages
> - Sauvegarde et restauration de fichiers
> - Arrêt des contrôleurs de domaine
> - Formatage de disques
> - Gestion des tâches planifiées
> 
> **Ne peut PAS** :
> - Modifier les appartenances aux groupes d'administration
> - Modifier les comptes de domaine

**Usage typique :** Techniciens gérant les serveurs sans être administrateurs complets.

#### Account Operators

> [!info] Account Operators
> **Portée** : Gestion limitée des comptes utilisateurs et groupes
> 
> **Capacités** :
> - Créer/modifier/supprimer utilisateurs et groupes (sauf groupes protégés)
> - Connexion locale sur contrôleurs de domaine
> 
> **Ne peut PAS modifier** :
> - Administrators
> - Domain Admins
> - Enterprise Admins
> - Schema Admins
> - Server Operators
> - Account Operators eux-mêmes

```powershell
# Ajouter un utilisateur au groupe Account Operators
Add-ADGroupMember -Identity "Account Operators" -Members "ADM-rh-manager"
```

**Usage typique :** Personnel RH gérant les comptes utilisateurs quotidiennement.

#### Backup Operators

> [!info] Backup Operators
> **Portée** : Droits de sauvegarde et restauration
> 
> **Capacités** :
> - Sauvegarde et restauration de tous les fichiers (même sans permission)
> - Connexion locale sur contrôleurs de domaine
> - Arrêt de serveurs
> 
> **Utilisé pour** :
> - Logiciels de sauvegarde
> - Agents de backup sur les serveurs

```powershell
# Ajouter un compte de service backup
Add-ADGroupMember -Identity "Backup Operators" -Members "SVC-Veeam-Backup"
```

#### Print Operators

> [!info] Print Operators
> **Portée** : Gestion complète des imprimantes
> 
> **Capacités** :
> - Installation/configuration/suppression d'imprimantes
> - Gestion des serveurs d'impression
> - Connexion locale sur contrôleurs de domaine

**Usage typique :** Équipe support gérant le parc d'imprimantes.

### Groupes Utilisateurs Standard

#### Domain Users

> [!info] Domain Users
> **Portée** : Tous les utilisateurs du domaine
> 
> **Caractéristiques** :
> - Groupe **Global**
> - Chaque utilisateur créé y est automatiquement ajouté
> - Groupe principal par défaut de tous les utilisateurs
> - Utilisé pour permissions de base accordées à tous

```powershell
# Vérifier qu'un utilisateur est bien dans Domain Users
Get-ADUser "jdupont" -Properties MemberOf | Select-Object -ExpandProperty MemberOf

# Nombre total d'utilisateurs dans le domaine
(Get-ADGroupMember -Identity "Domain Users" -Recursive).Count
```

**Usage :** Appliquer des permissions ou GPO à tous les utilisateurs du domaine.

#### Domain Computers

> [!info] Domain Computers
> **Portée** : Tous les ordinateurs du domaine
> 
> **Caractéristiques** :
> - Groupe **Global**
> - Chaque ordinateur joint au domaine y est ajouté automatiquement
> - Utilisé pour GPO et permissions applicables à tous les PC

```powershell
# Lister tous les ordinateurs du domaine
Get-ADGroupMember -Identity "Domain Computers" | Select-Object Name
```

#### Domain Guests

> [!info] Domain Guests
> **Portée** : Comptes invités du domaine
> 
> **Membres par défaut** :
> - Guest (compte intégré, désactivé par défaut)
> 
> **Usage** : Accès très limité pour visiteurs temporaires (rarement utilisé).

### Groupes de Contrôleurs de Domaine

#### Domain Controllers

> [!info] Domain Controllers
> **Portée** : Tous les contrôleurs de domaine
> 
> **Caractéristiques** :
> - Groupe **Global**
> - Contient tous les comptes ordinateurs des DC
> - Utilisé pour GPO spécifiques aux contrôleurs de domaine

```powershell
# Lister tous les DC du domaine
Get-ADGroupMember -Identity "Domain Controllers" | Select-Object Name
```

#### Read-only Domain Controllers (RODC)

> [!info] Read-only Domain Controllers
> **Portée** : Contrôleurs de domaine en lecture seule
> 
> **Caractéristiques** :
> - Contient les comptes des RODC
> - Utilisé pour stratégies spécifiques aux RODC (sites distants, peu sécurisés)

### Groupes Spéciaux de Sécurité

#### Protected Users

> [!info] Protected Users (Windows Server 2012 R2+)
> **Portée** : Protection renforcée pour comptes sensibles
> 
> **Protections appliquées** :
> - Désactivation de l'authentification NTLM
> - Désactivation de DES et RC4 dans Kerberos
> - Pas de mise en cache des credentials
> - Durée TGT Kerberos limitée à 4 heures
> - Impossible de déléguer (CredSSP)
> 
> **Usage recommandé** :
> - Comptes administrateurs de haut niveau
> - Comptes VIP nécessitant protection maximale

```powershell
# Ajouter un compte admin au groupe Protected Users
Add-ADGroupMember -Identity "Protected Users" -Members "ADM-ciso"

# Vérifier les membres
Get-ADGroupMember -Identity "Protected Users"
```

> [!warning] Attention aux Incompatibilités
> Protected Users peut casser certaines applications anciennes qui dépendent de :
> - NTLM
> - Kerberos RC4
> - Credentials en cache
> 
> Testez toujours avant de déployer sur comptes de production !

#### Denied RODC Password Replication Group

> [!info] Denied RODC Password Replication Group
> **Portée** : Comptes dont le mot de passe NE DOIT JAMAIS être répliqué sur RODC
> 
> **Membres par défaut** :
> - Administrators
> - Domain Admins
> - Enterprise Admins
> - Schema Admins
> - Backup Operators
> - Server Operators
> - Account Operators
> - Comptes de service critiques
> 
> **Objectif** : Sécurité des sites distants avec RODC

```powershell
# Ajouter un compte sensible
Add-ADGroupMember -Identity "Denied RODC Password Replication Group" -Members "SVC-Critical-App"
```

### Tableau Récapitulatif des Groupes Intégrés

| Groupe | Étendue | Capacités Clés | Usage Recommandé |
|--------|---------|----------------|------------------|
| **Administrators** | Domain Local | Contrôle total du domaine | Comptes admin uniquement |
| **Domain Admins** | Global | Admin domaine complet | 5-10 personnes max |
| **Enterprise Admins** | Universal | Admin de la forêt | Vide sauf lors d'opérations |
| **Schema Admins** | Universal | Modification schéma | Vide sauf lors d'opérations |
| **Server Operators** | Domain Local | Gestion serveurs limitée | Techniciens serveurs |
| **Account Operators** | Domain Local | Gestion utilisateurs limitée | Personnel RH |
| **Backup Operators** | Domain Local | Backup/restore | Comptes de service backup |
| **Print Operators** | Domain Local | Gestion imprimantes | Support imprimantes |
| **Domain Users** | Global | Tous les utilisateurs | GPO utilisateurs |
| **Domain Computers** | Global | Tous les ordinateurs | GPO ordinateurs |
| **Protected Users** | Global | Sécurité renforcée | Comptes VIP/Admin |

### Audit et Sécurité des Groupes Intégrés

```powershell
# Script d'audit des groupes sensibles
$sensitiveGroups = @(
    "Administrators",
    "Domain Admins",
    "Enterprise Admins",
    "Schema Admins",
    "Account Operators",
    "Backup Operators",
    "Server Operators"
)

foreach ($group in $sensitiveGroups) {
    Write-Host "
=== $group ===" -ForegroundColor Cyan
    try {
        $members = Get-ADGroupMember -Identity $group -ErrorAction Stop
        if ($members) {
            $members | Select-Object Name, SamAccountName, objectClass | Format-Table
        } else {
            Write-Host "Aucun membre" -ForegroundColor Green
        }
    } catch {
        Write-Host "Erreur : $($_.Exception.Message)" -ForegroundColor Red
    }
}

# Exporter vers CSV pour audit
$auditDate = Get-Date -Format "yyyyMMdd"
$reportPath = "C:\Audit\GroupesSensibles_$auditDate.csv"

$allMembers = @()
foreach ($group in $sensitiveGroups) {
    Get-ADGroupMember -Identity $group -ErrorAction SilentlyContinue | ForEach-Object {
        $allMembers += [PSCustomObject]@{
            Group = $group
            MemberName = $_.Name
            MemberType = $_.objectClass
            SamAccountName = $_.SamAccountName
        }
    }
}

$allMembers | Export-Csv -Path $reportPath -NoTypeInformation -Encoding UTF8
Write-Host "Rapport d'audit exporté : $reportPath" -ForegroundColor Green
```

> [!tip] Recommandations de Sécurité
> **Audit régulier :**
> - Mensuel pour groupes très sensibles (Domain Admins, Enterprise Admins)
> - Trimestriel pour autres groupes administratifs
> - Automatiser les alertes sur modifications
> 
> **Principe de moindre privilège :**
> - Privilégier des groupes moins puissants quand possible
> - Utiliser l'appartenance temporaire (JIT - Just In Time)
> - Documenter chaque ajout avec justification
> 
> **Surveillance :**
> - Activer l'audit AD pour changements de groupes
> - Alertes en temps réel sur modifications sensibles
> - Revue annuelle complète avec validation métier

---

## 🎯 Récapitulatif

> [!success] Points Clés à Retenir
> 
> **Utilisateurs :**
> - Créer via PowerShell pour standardisation et traçabilité
> - **UPN** : Format moderne, flexible (1024 car.), portée forêt
> - **SamAccountName** : Format hérité, limité (20 car.), portée domaine
> - Utiliser des suffixes UPN alternatifs pour expérience utilisateur optimale
> 
> **Groupes :**
> - **Sécurité** : Permissions + distribution (par défaut)
> - **Distribution** : Messagerie uniquement
> - Étendue détermine portée et contenu possibles
> - Stratégie AGDLP pour environnements multi-domaines
> 
> **Étendues :**
> - **Domain Local** : Permissions locales, membres de partout
> - **Global** : Utilisateurs d'un domaine, utilisable partout
> - **Universal** : Maximum flexibilité, impact réplication
> 
> **Conventions de nommage :**
> - Cohérence et clarté essentielles
> - Préfixes indicateurs (GRP, DL, SVC, ADM)
> - Documentation obligatoire
> 
> **Groupes intégrés :**
> - Utiliser avec parcimonie et précaution
> - Auditer régulièrement les membres
> - Privilégier moindre privilège
> - Groupes Enterprise/Schema Admins : vides par défaut

---

> [!quote] Bonne Pratique Finale
> **"Un Active Directory bien organisé commence par une stratégie claire de gestion des identités et des groupes. Investissez du temps dans la conception de vos conventions de nommage et de vos stratégies d'appartenance aux groupes : c'est un investissement qui se rentabilise sur la durée de vie de votre infrastructure."**
