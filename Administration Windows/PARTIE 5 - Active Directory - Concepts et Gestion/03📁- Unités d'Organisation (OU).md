
## 📋 Table des matières

1. [Concept des OU](#concept-des-ou)
2. [Création et structure d'OU](#création-et-structure-dou)
3. [Délégation de contrôle](#délégation-de-contrôle)
4. [Bonnes pratiques d'organisation](#bonnes-pratiques-dorganisation)
5. [OU vs groupes de sécurité](#ou-vs-groupes-de-sécurité)

---

## Concept des OU

### 📖 Qu'est-ce qu'une Unité d'Organisation ?

Une **Unité d'Organisation (Organizational Unit - OU)** est un conteneur dans Active Directory qui permet d'organiser et de regrouper des objets (utilisateurs, ordinateurs, groupes, autres OU) selon une structure logique.

> [!info] Conteneur vs OU
> Techniquement, une OU est un type spécial de conteneur AD. Les conteneurs par défaut (Users, Computers, Builtin) existent aussi mais ne peuvent pas recevoir de GPO ni être renommés/déplacés. Les OU offrent beaucoup plus de flexibilité.

### 🎯 Objectifs des OU

Les Unités d'Organisation servent trois objectifs principaux :

**1. Organisation logique**
- Structurer l'annuaire de manière cohérente
- Faciliter la navigation et la recherche
- Refléter la structure de l'entreprise

**2. Application des stratégies de groupe (GPO)**
- Appliquer des configurations spécifiques à des groupes d'objets
- Gérer les paramètres de sécurité par département
- Déployer des applications de manière ciblée

**3. Délégation d'administration**
- Confier des droits d'administration limités
- Permettre aux départements de gérer leurs propres utilisateurs
- Réduire le besoin de comptes administrateurs globaux

> [!tip] Triple fonction
> Contrairement aux groupes qui ne servent qu'à gérer les permissions, les OU combinent organisation, application de stratégies et délégation administrative. C'est leur force unique.

### 🏗️ Caractéristiques des OU

| Caractéristique | Description | Exemple |
|-----------------|-------------|---------|
| **Hiérarchique** | Peut contenir d'autres OU (structure en arbre) | RH → RH-Paris → RH-Paris-Recrutement |
| **Flexible** | Peut être renommée, déplacée, supprimée | Renommer "IT" en "Informatique" |
| **Héritage** | Les GPO s'appliquent aux OU enfants | GPO sur "Utilisateurs" s'applique à toutes les sous-OU |
| **Délégable** | Droits administratifs assignables | Le manager RH gère l'OU RH uniquement |
| **Protégeable** | Protection contre la suppression accidentelle | Empêche la suppression de l'OU "Production" |

### 📊 Types d'objets dans une OU

Une OU peut contenir différents types d'objets :

```
OU: Marketing
├── 👤 Utilisateurs (User accounts)
│   ├── Jean Dupont
│   ├── Marie Martin
│   └── Pierre Durand
├── 🖥️ Ordinateurs (Computer accounts)
│   ├── PC-MARKETING-01
│   ├── PC-MARKETING-02
│   └── LAPTOP-MKT-03
├── 👥 Groupes (Security & Distribution groups)
│   ├── GRP-Marketing-RW
│   ├── GRP-Marketing-Managers
│   └── DL-Marketing-Team
├── 🖨️ Ressources (Printers, Shared folders)
│   └── PRINTER-MKT-COLOR
└── 📁 Sous-OU
    ├── OU: Marketing-Digital
    └── OU: Marketing-Communication
```

### 🔍 OU vs Conteneurs par défaut

Active Directory possède des conteneurs par défaut créés automatiquement :

| Conteneur | Description | Peut recevoir GPO ? | Modifiable ? |
|-----------|-------------|---------------------|--------------|
| **Users** | Utilisateurs par défaut | ❌ Non | ❌ Non |
| **Computers** | Ordinateurs par défaut | ❌ Non | ❌ Non |
| **Builtin** | Groupes système | ❌ Non | ❌ Non |
| **Domain Controllers** | OU des DC (exception) | ✅ Oui | ⚠️ Limité |
| **OU personnalisée** | Créée par l'admin | ✅ Oui | ✅ Oui |

> [!warning] Conteneurs par défaut
> Les conteneurs par défaut "Users" et "Computers" ne peuvent PAS recevoir de GPO. C'est une limitation majeure. Il est fortement recommandé de créer vos propres OU et d'y déplacer les objets.

**Rediriger les créations par défaut vers des OU personnalisées** :
```powershell
# Rediriger les nouveaux utilisateurs vers une OU personnalisée
redirusr "OU=Nouveaux-Utilisateurs,DC=entreprise,DC=internal"

# Rediriger les nouveaux ordinateurs vers une OU personnalisée
redircmp "OU=Nouveaux-Ordinateurs,DC=entreprise,DC=internal"

# Vérifier les redirections actuelles
Get-ADDomain | Select-Object UsersContainer, ComputersContainer
```

### 🌳 Chemin distinctif (Distinguished Name)

Chaque OU possède un **DN (Distinguished Name)** unique qui représente son emplacement complet dans l'arborescence AD.

**Format du DN** :
```
OU=nom,OU=parent,DC=domaine,DC=tld
```

**Exemples** :
```ldap
# OU racine
OU=Utilisateurs,DC=entreprise,DC=internal

# OU imbriquée (2 niveaux)
OU=Paris,OU=Utilisateurs,DC=entreprise,DC=internal

# OU profondément imbriquée (3 niveaux)
OU=Recrutement,OU=RH,OU=Utilisateurs,DC=entreprise,DC=internal
```

> [!info] Lecture du DN
> Le DN se lit de droite à gauche (du plus général au plus spécifique) :
> - Domaine : `DC=entreprise,DC=internal`
> - Puis : `OU=Utilisateurs`
> - Puis : `OU=RH`
> - Enfin : `OU=Recrutement`

**Obtenir le DN d'une OU** :
```powershell
# Rechercher une OU par son nom
Get-ADOrganizationalUnit -Filter 'Name -eq "Marketing"' | Select-Object Name, DistinguishedName

# Résultat exemple :
# Name       DistinguishedName
# ----       -----------------
# Marketing  OU=Marketing,OU=Utilisateurs,DC=entreprise,DC=internal
```

---

## Création et structure d'OU

### 🏗️ Planification de la structure

Avant de créer des OU, il est crucial de planifier une structure cohérente et durable.

**Questions à se poser** :

1. **Structure organisationnelle** :
   - Comment l'entreprise est-elle organisée ?
   - Par département ? Par géographie ? Par fonction ?
   - Y a-t-il des filiales ou divisions distinctes ?

2. **Besoins en GPO** :
   - Quels groupes d'utilisateurs ont besoin de paramètres différents ?
   - Y a-t-il des ordinateurs avec des configurations spéciales ?
   - Les stratégies varient-elles par localisation ?

3. **Délégation administrative** :
   - Quels départements gèrent leurs propres utilisateurs ?
   - Y a-t-il des administrateurs locaux par site ?
   - Qui doit avoir accès à quelles ressources ?

> [!tip] KISS Principle
> Keep It Simple, Stupid ! Une structure trop complexe devient ingérable. Visez 3-4 niveaux maximum de profondeur d'OU. Au-delà, envisagez de revoir votre conception.

### 📐 Modèles de structuration courants

#### Modèle 1 : Par département (fonctionnel)

Recommandé pour les petites et moyennes entreprises avec un site unique.

```
entreprise.internal
└── Utilisateurs
    ├── Direction
    ├── RH
    ├── IT
    ├── Marketing
    ├── Ventes
    └── Finance
└── Ordinateurs
    ├── Laptops
    ├── Desktops
    └── Serveurs
└── Groupes
    └── Securite
└── Ressources
```

**Avantages** :
- Simple et intuitif
- Facilite la délégation par département
- GPO faciles à appliquer par fonction

**Inconvénients** :
- Difficile pour les organisations multi-sites
- Complexe si les politiques varient par localisation

#### Modèle 2 : Par géographie

Recommandé pour les entreprises multi-sites avec des besoins locaux spécifiques.

```
entreprise.internal
└── Paris
    ├── Utilisateurs
    ├── Ordinateurs
    └── Groupes
└── Londres
    ├── Utilisateurs
    ├── Ordinateurs
    └── Groupes
└── Tokyo
    ├── Utilisateurs
    ├── Ordinateurs
    └── Groupes
```

**Avantages** :
- Facilite la gestion par site
- Adapté aux fuseaux horaires différents
- Simplifie la délégation aux administrateurs locaux

**Inconvénients** :
- Duplication des GPO entre sites
- Difficile pour les utilisateurs mobiles

#### Modèle 3 : Hybride (géographie + fonction)

Recommandé pour les grandes entreprises complexes.

```
entreprise.internal
└── Utilisateurs
    ├── Paris
    │   ├── Direction
    │   ├── IT
    │   └── Marketing
    ├── Londres
    │   ├── Direction
    │   ├── IT
    │   └── Ventes
    └── Tokyo
        ├── IT
        └── Support
└── Ordinateurs
    ├── Paris
    │   ├── Laptops
    │   └── Desktops
    ├── Londres
    └── Tokyo
```

**Avantages** :
- Flexibilité maximale
- Adapté aux grandes organisations
- Permet une granularité fine

**Inconvénients** :
- Plus complexe à gérer
- Risque de structure trop profonde
- Demande plus de planification

#### Modèle 4 : Par type d'objet (technique)

Structure simple basée sur le type d'objet.

```
entreprise.internal
├── Users-OU
│   └── (tous les utilisateurs, possiblement avec sous-OU)
├── Computers-OU
│   ├── Workstations
│   ├── Laptops
│   └── Servers
├── Groups-OU
│   ├── Security-Groups
│   └── Distribution-Groups
└── Service-Accounts-OU
```

**Avantages** :
- Très simple
- Facile à comprendre
- Bon pour les petites structures

**Inconvénients** :
- Manque de granularité
- Difficile d'appliquer des GPO ciblées

> [!warning] Éviter les structures trop profondes
> Plus de 5 niveaux de profondeur devient difficile à gérer. Exemple à éviter :
> `OU=Stagiaires,OU=Junior,OU=Recrutement,OU=RH,OU=Paris,OU=France,OU=Europe,OU=Utilisateurs`

### 🛠️ Création d'OU via GUI

**Méthode : Active Directory Users and Computers (ADUC)**

1. Ouvrir **Active Directory Users and Computers** (`dsa.msc`)
2. Naviguer vers l'emplacement parent (ex: domaine racine)
3. Clic droit → **New** → **Organizational Unit**
4. Entrer le nom de l'OU (ex: "Utilisateurs")
5. Cocher **Protect container from accidental deletion** (recommandé)
6. Cliquer sur **OK**

**Création d'OU imbriquée** :
1. Clic droit sur l'OU parente (ex: "Utilisateurs")
2. **New** → **Organizational Unit**
3. Nom : "RH"
4. **OK**

> [!tip] Protection contre la suppression
> Activez TOUJOURS "Protect container from accidental deletion" pour les OU importantes. Cela empêche une suppression accidentelle qui pourrait affecter des centaines d'objets.

### 💻 Création d'OU via PowerShell

PowerShell est plus rapide et reproductible, surtout pour créer plusieurs OU.

**Créer une OU simple** :
```powershell
# Créer une OU à la racine du domaine
New-ADOrganizationalUnit -Name "Utilisateurs" -Path "DC=entreprise,DC=internal" -ProtectedFromAccidentalDeletion $true

# Vérifier la création
Get-ADOrganizationalUnit -Filter 'Name -eq "Utilisateurs"'
```

**Créer une OU imbriquée** :
```powershell
# Créer une OU enfant dans une OU existante
New-ADOrganizationalUnit -Name "RH" -Path "OU=Utilisateurs,DC=entreprise,DC=internal" -ProtectedFromAccidentalDeletion $true
```

**Créer plusieurs OU d'un coup** :
```powershell
# Définir les OU à créer
$OUs = @("Direction", "RH", "IT", "Marketing", "Ventes", "Finance")

# Créer chaque OU dans l'OU "Utilisateurs"
foreach ($OU in $OUs) {
    New-ADOrganizationalUnit -Name $OU -Path "OU=Utilisateurs,DC=entreprise,DC=internal" -ProtectedFromAccidentalDeletion $true
    Write-Host "OU $OU créée avec succès" -ForegroundColor Green
}
```

**Créer une structure complète** :
```powershell
# Script pour créer une structure complète
$Domain = "DC=entreprise,DC=internal"

# OU de premier niveau
$FirstLevelOUs = @("Utilisateurs", "Ordinateurs", "Groupes", "Ressources")

foreach ($OU in $FirstLevelOUs) {
    New-ADOrganizationalUnit -Name $OU -Path $Domain -ProtectedFromAccidentalDeletion $true
}

# OU de second niveau sous "Utilisateurs"
$UserOUs = @("Direction", "RH", "IT", "Marketing", "Ventes", "Finance")

foreach ($OU in $UserOUs) {
    New-ADOrganizationalUnit -Name $OU -Path "OU=Utilisateurs,$Domain" -ProtectedFromAccidentalDeletion $true
}

# OU de second niveau sous "Ordinateurs"
$ComputerOUs = @("Laptops", "Desktops", "Serveurs")

foreach ($OU in $ComputerOUs) {
    New-ADOrganizationalUnit -Name $OU -Path "OU=Ordinateurs,$Domain" -ProtectedFromAccidentalDeletion $true
}

Write-Host "Structure OU créée avec succès !" -ForegroundColor Green
```

### 📝 Propriétés d'une OU

Chaque OU possède plusieurs propriétés configurables.

**Propriétés principales** :

| Propriété | Description | Modifiable |
|-----------|-------------|------------|
| **Name** | Nom de l'OU | ✅ Oui |
| **Description** | Description textuelle | ✅ Oui |
| **ProtectedFromAccidentalDeletion** | Protection suppression | ✅ Oui |
| **ManagedBy** | Responsable/Gestionnaire | ✅ Oui |
| **DistinguishedName** | Chemin complet | ❌ Non (auto) |
| **Created** | Date de création | ❌ Non (auto) |
| **Modified** | Date de modification | ❌ Non (auto) |

**Modifier les propriétés via PowerShell** :
```powershell
# Ajouter une description
Set-ADOrganizationalUnit -Identity "OU=RH,OU=Utilisateurs,DC=entreprise,DC=internal" -Description "Département Ressources Humaines"

# Définir un gestionnaire (ManagedBy)
Set-ADOrganizationalUnit -Identity "OU=RH,OU=Utilisateurs,DC=entreprise,DC=internal" -ManagedBy "CN=Marie Martin,OU=RH,OU=Utilisateurs,DC=entreprise,DC=internal"

# Désactiver la protection (pour permettre la suppression)
Set-ADOrganizationalUnit -Identity "OU=Test,DC=entreprise,DC=internal" -ProtectedFromAccidentalDeletion $false
```

### 🔄 Déplacer des objets vers une OU

**Déplacer un utilisateur via GUI** :
1. Ouvrir ADUC
2. Localiser l'utilisateur
3. Clic droit → **Move**
4. Sélectionner l'OU de destination
5. **OK**

**Déplacer un utilisateur via PowerShell** :
```powershell
# Déplacer un utilisateur
Move-ADObject -Identity "CN=Jean Dupont,CN=Users,DC=entreprise,DC=internal" -TargetPath "OU=RH,OU=Utilisateurs,DC=entreprise,DC=internal"

# Déplacer un ordinateur
Move-ADObject -Identity "CN=PC-MKT-01,CN=Computers,DC=entreprise,DC=internal" -TargetPath "OU=Laptops,OU=Ordinateurs,DC=entreprise,DC=internal"
```

**Déplacer plusieurs utilisateurs d'un coup** :
```powershell
# Déplacer tous les utilisateurs d'un département spécifique
Get-ADUser -Filter 'Department -eq "Marketing"' | Move-ADObject -TargetPath "OU=Marketing,OU=Utilisateurs,DC=entreprise,DC=internal"

# Déplacer tous les ordinateurs portables (dont le nom commence par LAPTOP)
Get-ADComputer -Filter 'Name -like "LAPTOP*"' | Move-ADObject -TargetPath "OU=Laptops,OU=Ordinateurs,DC=entreprise,DC=internal"
```

### 🗑️ Suppression d'OU

> [!warning] Suppression = perte de données
> Supprimer une OU supprime également tous les objets qu'elle contient (utilisateurs, ordinateurs, groupes, sous-OU). Cette action est irréversible (sauf restauration depuis la corbeille AD ou sauvegarde).

**Supprimer une OU protégée** :
```powershell
# Étape 1 : Retirer la protection
Set-ADOrganizationalUnit -Identity "OU=Test,DC=entreprise,DC=internal" -ProtectedFromAccidentalDeletion $false

# Étape 2 : Supprimer l'OU (et tout son contenu !)
Remove-ADOrganizationalUnit -Identity "OU=Test,DC=entreprise,DC=internal" -Confirm:$false
```

**Supprimer une OU récursivement (avec tous ses enfants)** :
```powershell
# Attention : supprime TOUT (OU + tous les objets enfants)
Remove-ADOrganizationalUnit -Identity "OU=OldStructure,DC=entreprise,DC=internal" -Recursive -Confirm:$false
```

> [!tip] Sauvegarde avant suppression
> Avant de supprimer une OU importante, exportez son contenu :
> ```powershell
> Get-ADObject -SearchBase "OU=ToDelete,DC=entreprise,DC=internal" -Filter * | Export-Csv -Path "C:\Backup\OU-ToDelete-Backup.csv" -NoTypeInformation
> ```

---

## Délégation de contrôle

La délégation de contrôle permet de confier des droits administratifs limités sur une OU spécifique à des utilisateurs ou groupes sans leur donner des privilèges globaux sur tout le domaine.

### 🎯 Pourquoi déléguer ?

**Avantages de la délégation** :

✅ **Sécurité renforcée** :
- Principe du moindre privilège
- Limite l'impact des comptes compromis
- Réduit le nombre d'administrateurs de domaine

✅ **Efficacité opérationnelle** :
- Les départements gèrent leurs propres utilisateurs
- Réduction de la charge de travail pour l'équipe IT centrale
- Réponse plus rapide aux demandes locales

✅ **Responsabilisation** :
- Les managers ont le contrôle sur leurs équipes
- Meilleure appropriation des processus
- Autonomie des départements

> [!example] Scénario typique
> Le responsable RH a besoin de créer et modifier des comptes utilisateurs pour son département, mais ne doit PAS pouvoir toucher aux comptes IT ou Finance. La délégation permet de lui donner exactement ces droits, ni plus, ni moins.

### 📋 Types de droits délégables

Vous pouvez déléguer différents niveaux de contrôle :

| Droit | Description | Cas d'usage |
|-------|-------------|-------------|
| **Créer des utilisateurs** | Créer de nouveaux comptes | Équipe RH pour les nouveaux employés |
| **Modifier des utilisateurs** | Changer attributs (téléphone, adresse) | Managers pour leurs équipes |
| **Réinitialiser les mots de passe** | Débloquer et changer mots de passe | Help desk de premier niveau |
| **Gérer l'appartenance aux groupes** | Ajouter/retirer des membres | Responsables de projets |
| **Créer/supprimer des ordinateurs** | Gérer les comptes machines | Équipe desktop support |
| **Gérer les GPO** | Créer et lier des GPO | Administrateurs de site |
| **Contrôle total sur l'OU** | Tous les droits sur l'OU | Administrateur délégué de département |

### 🧙 Délégation via l'Assistant (GUI)

**Méthode : Delegation of Control Wizard**

1. Ouvrir **Active Directory Users and Computers**
2. Clic droit sur l'OU cible → **Delegate Control**
3. **Welcome** → Next

4. **Users or Groups** :
   - Cliquer sur **Add**
   - Sélectionner les utilisateurs/groupes à qui déléguer
   - Exemple : "GRP-RH-Admins"
   - **OK** → Next

5. **Tasks to Delegate** :
   - Cocher les tâches à déléguer :
     - ☑ Create, delete, and manage user accounts
     - ☑ Reset user passwords and force password change at next logon
     - ☑ Read all user information
     - ☑ Modify the membership of a group
   - **Next**

6. **Review** → **Finish**

> [!info] Tâches communes
> L'assistant propose des tâches prédéfinies courantes. Pour des besoins plus spécifiques, sélectionnez "Create a custom task to delegate".

**Délégation personnalisée** :

Pour une granularité fine :

1. **Tasks to Delegate** → Sélectionner **Create a custom task to delegate**
2. **Active Directory Object Type** :
   - Choisir le type d'objet : User objects, Computer objects, Group objects, etc.
3. **Permissions** :
   - Cocher les permissions spécifiques (Read, Write, Create, Delete)
   - Exemple : Read + Write sur les propriétés "telephoneNumber" et "mail"

### 💻 Délégation via PowerShell (DSACLS)

Pour une automatisation et une précision maximale, utilisez `dsacls` (Directory Service ACLs).

**Syntaxe de base** :
```powershell
dsacls "DN_de_l'OU" /G "utilisateur_ou_groupe:permissions"
```

**Exemples pratiques** :

**1. Permettre la réinitialisation des mots de passe** :
```powershell
# Déléguer la réinitialisation de mots de passe sur l'OU RH au groupe Help-Desk
dsacls "OU=RH,OU=Utilisateurs,DC=entreprise,DC=internal" /G "ENTREPRISE\GRP-HelpDesk:CA;Reset Password;user"

# CA = Control Access (droit spécial)
# "Reset Password" = nom de la permission étendue
# "user" = s'applique aux objets utilisateur
```

**2. Permettre la création d'utilisateurs** :
```powershell
# Déléguer la création d'utilisateurs au groupe RH-Admins
dsacls "OU=RH,OU=Utilisateurs,DC=entreprise,DC=internal" /G "ENTREPRISE\GRP-RH-Admins:CC;user"

# CC = Create Child (créer des objets enfants)
# "user" = type d'objet utilisateur
```

**3. Permettre la modification de propriétés spécifiques** :
```powershell
# Permettre la modification du numéro de téléphone et email uniquement
dsacls "OU=RH,OU=Utilisateurs,DC=entreprise,DC=internal" /G "ENTREPRISE\GRP-RH-Admins:WP;telephoneNumber;user"
dsacls "OU=RH,OU=Utilisateurs,DC=entreprise,DC=internal" /G "ENTREPRISE\GRP-RH-Admins:WP;mail;user"

# WP = Write Property (écrire une propriété)
```

**4. Contrôle total sur une OU** :
```powershell
# Donner un contrôle total au groupe Marketing-IT sur l'OU Marketing
dsacls "OU=Marketing,OU=Utilisateurs,DC=entreprise,DC=internal" /G "ENTREPRISE\GRP-Marketing-IT:GA"

# GA = Generic All (tous les droits)
```

**Permissions courantes dans dsacls** :

| Code | Signification | Description |
|------|---------------|-------------|
| **GA** | Generic All | Contrôle total |
| **GR** | Generic Read | Lecture complète |
| **GW** | Generic Write | Écriture complète |
| **CC** | Create Child | Créer des objets enfants |
| **DC** | Delete Child | Supprimer des objets enfants |
| **WP** | Write Property | Écrire une propriété |
| **RP** | Read Property | Lire une propriété |
| **CA** | Control Access | Droits d'accès étendus |

### 🔍 Vérifier les délégations existantes

**Via ADUC (GUI)** :
1. Clic droit sur l'OU → **Properties**
2. Onglet **Security**
3. Cliquer sur **Advanced**
4. Examiner la liste des ACE (Access Control Entries)

> [!tip] Afficher l'onglet Security
> Par défaut, l'onglet Security n'est pas visible. Activez-le : ADUC → Menu **View** → Cocher **Advanced Features**

**Via PowerShell** :
```powershell
# Afficher les ACL d'une OU
(Get-Acl "AD:\OU=RH,OU=Utilisateurs,DC=entreprise,DC=internal").Access | 
    Select-Object IdentityReference, ActiveDirectoryRights, AccessControlType | 
    Format-Table -AutoSize

# Filtrer pour voir uniquement les délégations personnalisées (non héritées)
(Get-Acl "AD:\OU=RH,OU=Utilisateurs,DC=entreprise,DC=internal").Access | 
    Where-Object {$_.IsInherited -eq $false} | 
    Select-Object IdentityReference, ActiveDirectoryRights, AccessControlType
```

**Via dsacls** :
```powershell
# Afficher toutes les permissions sur une OU
dsacls "OU=RH,OU=Utilisateurs,DC=entreprise,DC=internal"

# Résultat : liste détaillée de toutes les ACE avec trustee et permissions
```

### 🗑️ Retirer une délégation

**Via ADUC** :
1. Clic droit sur l'OU → **Properties** → **Security** → **Advanced**
2. Sélectionner l'entrée de permission à retirer
3. Cliquer sur **Remove**
4. **OK** → **Apply** → **OK**

**Via dsacls** :
```powershell
# Retirer une permission spécifique
dsacls "OU=RH,OU=Utilisateurs,DC=entreprise,DC=internal" /R "ENTREPRISE\GRP-RH-Admins"

# /R = Remove (retire toutes les permissions pour ce trustee)
```

**Via PowerShell (méthode ACL)** :
```powershell
# Obtenir l'ACL actuelle
$OU = "OU=RH,OU=Utilisateurs,DC=entreprise,DC=internal"
$ACL = Get-Acl "AD:\$OU"

# Identifier l'ACE à retirer
$Identity = "ENTREPRISE\GRP-RH-Admins"
$ACEtoRemove = $ACL.Access | Where-Object {$_.IdentityReference -eq $Identity}

# Retirer l'ACE
foreach ($ACE in $ACEtoRemove) {
    $ACL.RemoveAccessRule($ACE) | Out-Null
}

# Appliquer les modifications
Set-Acl "AD:\$OU" -AclObject $ACL
```

### 🛡️ Bonnes pratiques de délégation

**1. Utiliser des groupes, pas des utilisateurs individuels**
```powershell
# ✅ BON : Déléguer à un groupe
dsacls "OU=RH,DC=entreprise,DC=internal" /G "ENTREPRISE\GRP-RH-Admins:CC;user"

# ❌ MAUVAIS : Déléguer à un utilisateur individuel
dsacls "OU=RH,DC=entreprise,DC=internal" /G "ENTREPRISE\Marie.Martin:CC;user"
```

**2. Principe du moindre privilège**
- Ne donnez que les droits strictement nécessaires
- Évitez "Generic All" sauf si vraiment requis
- Préférez des permissions granulaires

**3. Documenter les délégations**
```powershell
# Exporter les délégations pour documentation
$OUs = Get-ADOrganizationalUnit -Filter *

$Report = foreach ($OU in $OUs) {
    (Get-Acl "AD:\$($OU.DistinguishedName)").Access | 
    Where-Object {$_.IsInherited -eq $false} |
    Select-Object @{N='OU';E={$OU.Name}}, 
                  IdentityReference, 
                  ActiveDirectoryRights, 
                  AccessControlType
}

$Report | Export-Csv "C:\Reports\OU-Delegations.csv" -NoTypeInformation
```

**4. Auditer régulièrement**
- Vérifiez les délégations tous les 6 mois
- Retirez les droits pour les utilisateurs qui ont changé de poste
- Vérifiez que les groupes utilisés sont encore appropriés

**5. Tester les délégations**
```powershell
# Se connecter avec un compte délégué et tester
# Exemple : se connecter comme membre du groupe RH-Admins et tenter de créer un utilisateur

# Vérifier si un utilisateur a des droits sur une OU
$User = "ENTREPRISE\Marie.Martin"
$OU = "OU=RH,OU=Utilisateurs,DC=entreprise,DC=internal"

(Get-Acl "AD:\$OU").Access | Where-Object {$_.IdentityReference -like "*$User*"}
```

### ⚠️ Pièges courants

**Piège 1 : Héritage non bloqué**
- Les permissions héritées du parent peuvent s'ajouter ou entrer en conflit
- Solution : Bien comprendre l'héritage ou le bloquer si nécessaire

**Piège 2 : Oublier de créer le groupe de délégation**
- Créez d'abord le groupe, puis déléguez
- Ajoutez les utilisateurs au groupe, pas l'inverse

**Piège 3 : Déléguer sur une OU vide**
- Si vous déplacez ensuite des objets, les permissions s'appliquent
- Testez avec quelques objets d'abord

**Piège 4 : Confondre délégation et appartenance au groupe**
- Être dans "Domain Admins" ≠ délégation ciblée
- La délégation offre un contrôle précis, pas un accès global

---

## Bonnes pratiques d'organisation

### 🏗️ Principes de conception

**1. Simplicité avant tout**

> [!tip] Règle des 3-4 niveaux
> Limitez la profondeur de votre structure à 3-4 niveaux maximum. Au-delà, la navigation devient complexe et les GPO difficiles à gérer.

```
✅ BON (3 niveaux) :
entreprise.internal
└── Utilisateurs
    └── Paris
        └── IT

❌ MAUVAIS (7 niveaux) :
entreprise.internal
└── Géographie
    └── Europe
        └── France
            └── Île-de-France
                └── Paris
                    └── Département
                        └── IT
```

**2. Cohérence et standardisation**

**Conventions de nommage** :

| Type | Convention | Exemple |
|------|------------|---------|
| **OU géographique** | Nom de la ville | Paris, Londres, Tokyo |
| **OU départementale** | Nom du département | RH, IT, Marketing |
| **OU technique** | Préfixe descriptif | SRV-Production, WKS-Laptops |
| **OU de ressources** | Préfixe RES- | RES-Imprimantes, RES-Partages |

```powershell
# ✅ COHÉRENT
OU=Paris, OU=Londres, OU=Tokyo

# ❌ INCOHÉRENT
OU=Paris, OU=London, OU=Bureau-Tokyo
```

**3. Planifier pour l'évolution**

Concevez votre structure en anticipant la croissance :
- Laissez de la place pour de nouveaux départements
- Prévoyez l'expansion géographique
- Évitez les structures trop rigides

> [!example] Structure évolutive
> ```
> Utilisateurs
> ├── Paris (peut accueillir de nouveaux départements)
> ├── Londres
> ├── [Future-Office]  ← Place pour expansion
> └── Remote-Users (pour les télétravailleurs)
> ```

### 📊 Organisation par type d'objet

**Séparation claire des types d'objets** :

```
entreprise.internal
├── Users-OU
│   ├── Employees
│   ├── Contractors
│   └── Service-Accounts
├── Computers-OU
│   ├── Workstations
│   ├── Laptops
│   └── Servers
├── Groups-OU
│   ├── Security-Groups
│   └── Distribution-Groups
└── Resources-OU
    ├── Printers
    └── Shared-Folders
```

**Avantages** :
- Clarté immédiate de l'organisation
- GPO faciles à cibler (ex: GPO sur Workstations seulement)
- Délégation simplifiée par type

### 🔄 Gestion des comptes spéciaux

**Comptes de service** :

Créez une OU dédiée pour les comptes de service (service accounts).

```powershell
# Structure pour les comptes de service
New-ADOrganizationalUnit -Name "Service-Accounts" -Path "OU=Users-OU,DC=entreprise,DC=internal" -ProtectedFromAccidentalDeletion $true

# Sous-OU par application
New-ADOrganizationalUnit -Name "SQL-Services" -Path "OU=Service-Accounts,OU=Users-OU,DC=entreprise,DC=internal"
New-ADOrganizationalUnit -Name "IIS-Services" -Path "OU=Service-Accounts,OU=Users-OU,DC=entreprise,DC=internal"
```

**Comptes désactivés/archivés** :

Ne supprimez pas immédiatement les comptes des employés partis.

```powershell
# OU pour les comptes désactivés
New-ADOrganizationalUnit -Name "Disabled-Accounts" -Path "DC=entreprise,DC=internal"

# Déplacer un utilisateur désactivé
Disable-ADAccount -Identity "Jean.Dupont"
Move-ADObject -Identity "CN=Jean Dupont,OU=RH,OU=Utilisateurs,DC=entreprise,DC=internal" -TargetPath "OU=Disabled-Accounts,DC=entreprise,DC=internal"
```

**Avantages** :
- Traçabilité (on sait qui a été désactivé)
- Récupération facile si erreur
- Audit simplifié
- Pas de pollution des OU actives

### 🎯 Organisation orientée GPO

Structurez vos OU en fonction de vos besoins en stratégies de groupe.

**Principe** : Regroupez ensemble les objets qui partagent les mêmes besoins en configuration.

```
Ordinateurs
├── Workstations-Standard
│   └── (GPO : Configuration bureautique standard)
├── Workstations-SecureZone
│   └── (GPO : Configuration renforcée + restrictions)
├── Laptops-Mobile
│   └── (GPO : VPN automatique + BitLocker)
└── Kiosks-Public
    └── (GPO : Verrouillage complet)
```

> [!tip] Une OU = Un ensemble de GPO cohérent
> Si deux groupes d'ordinateurs nécessitent des GPO radicalement différentes, ils méritent probablement des OU séparées.

### 🌍 Multi-site et réplication

Pour les organisations avec plusieurs sites géographiques :

**Structure recommandée** :

```
entreprise.internal
├── Site-Paris
│   ├── Users
│   ├── Computers
│   └── Groups
├── Site-Londres
│   ├── Users
│   ├── Computers
│   └── Groups
└── Site-Tokyo
    ├── Users
    ├── Computers
    └── Groups
```

**Avantages** :
- Facilite la délégation aux administrateurs locaux
- GPO spécifiques par site (fuseaux horaires, langues)
- Alignement avec les sites AD physiques
- Simplifie la gestion des bureaux régionaux

### 📝 Documentation de la structure

**Documentez votre structure OU** :

```powershell
# Script pour documenter la structure OU
$AllOUs = Get-ADOrganizationalUnit -Filter * | Select-Object Name, DistinguishedName, Description

# Exporter vers CSV
$AllOUs | Export-Csv "C:\Documentation\OU-Structure.csv" -NoTypeInformation

# Créer une vue hiérarchique
function Show-OUHierarchy {
    param($Parent = (Get-ADDomain).DistinguishedName, $Indent = 0)
    
    $OUs = Get-ADOrganizationalUnit -Filter * -SearchBase $Parent -SearchScope OneLevel
    
    foreach ($OU in $OUs) {
        Write-Host ("  " * $Indent) "└── $($OU.Name)"
        Show-OUHierarchy -Parent $OU.DistinguishedName -Indent ($Indent + 1)
    }
}

# Afficher la hiérarchie
Show-OUHierarchy
```

**Inclure dans la documentation** :
- Diagramme de la structure OU
- But de chaque OU principale
- GPO appliquées à chaque OU
- Délégations en place
- Responsables de chaque OU

### 🔒 Sécurité des OU

**1. Protection contre la suppression accidentelle**

```powershell
# Activer la protection sur toutes les OU
Get-ADOrganizationalUnit -Filter * | Set-ADOrganizationalUnit -ProtectedFromAccidentalDeletion $true

# Vérifier les OU non protégées
Get-ADOrganizationalUnit -Filter * -Properties ProtectedFromAccidentalDeletion | 
    Where-Object {$_.ProtectedFromAccidentalDeletion -eq $false} | 
    Select-Object Name, DistinguishedName
```

**2. Limitation de l'héritage des permissions**

Pour les OU sensibles, bloquez l'héritage si nécessaire :

```powershell
# Bloquer l'héritage sur une OU (via GUI recommandé)
# ADUC → OU Properties → Security → Advanced → Disable inheritance
```

> [!warning] Bloquer l'héritage avec précaution
> Bloquer l'héritage peut casser des délégations ou empêcher les administrateurs de domaine d'accéder à l'OU. À n'utiliser que pour des cas très spécifiques.

**3. Audit des modifications**

```powershell
# Activer l'audit des modifications sur les OU critiques
# Via GPO : Computer Configuration → Windows Settings → Security Settings → 
#           Advanced Audit Policy Configuration → DS Access → 
#           Audit Directory Service Changes
```

### 🚫 Erreurs courantes à éviter

**1. Structure trop plate**
```
❌ MAUVAIS :
entreprise.internal
├── Users
│   ├── Jean.Dupont
│   ├── Marie.Martin
│   ├── Pierre.Durand
│   └── ... (500 utilisateurs)
```
**Solution** : Créez des OU par département, site, ou fonction.

**2. Structure trop profonde**
```
❌ MAUVAIS :
OU=Stagiaires,OU=Temporaires,OU=NonPermanents,OU=Recrutement,OU=RH,OU=Paris,OU=France,OU=Europe,OU=Utilisateurs
```
**Solution** : Simplifiez à 3-4 niveaux maximum.

**3. Noms incohérents**
```
❌ MAUVAIS :
OU=Paris, OU=London, OU=Bureau-Tokyo, OU=NYC
```
**Solution** : Standardisez (tout en français ou tout en anglais).

**4. Ne pas utiliser les OU**
```
❌ MAUVAIS :
Tous les utilisateurs dans CN=Users (conteneur par défaut)
```
**Solution** : Créez des OU et déplacez-y les objets.

**5. OU basée uniquement sur la structure organisationnelle**

L'organigramme de l'entreprise change, mais la structure AD devrait rester stable.

```
❌ MAUVAIS :
OU=Manager-Jean,OU=Direction
(Si Jean part, faut-il renommer l'OU ?)
```

**Solution** : Basez-vous sur des fonctions, pas sur des personnes.

---

## OU vs groupes de sécurité

Une confusion courante pour les débutants : quand utiliser une OU et quand utiliser un groupe de sécurité ? Bien qu'ils puissent sembler similaires, ils ont des objectifs totalement différents.

### 🔍 Différences fondamentales

| Critère | OU (Organizational Unit) | Groupe de sécurité |
|---------|--------------------------|-------------------|
| **Objectif principal** | Organiser et structurer l'annuaire | Gérer les permissions d'accès |
| **Contenu** | Objets AD (users, computers, groupes, sous-OU) | Membres (users, computers, autres groupes) |
| **Hiérarchie** | Hiérarchique (arbre) | Plat (liste de membres) |
| **Application GPO** | ✅ Oui | ❌ Non (sauf filtrage) |
| **Permissions NTFS** | ❌ Non | ✅ Oui |
| **Délégation** | ✅ Oui | ⚠️ Limité |
| **Visible dans** | Active Directory | Active Directory + Explorateur de fichiers |
| **Modification** | Requiert droits AD | Peut être auto-géré |

### 📂 OU : Organisation et administration

**Utilisez une OU pour** :

✅ **Structurer l'annuaire Active Directory**
- Organiser les objets de manière logique
- Faciliter la navigation dans ADUC

✅ **Appliquer des stratégies de groupe (GPO)**
- Déployer des logiciels à un département
- Configurer des paramètres de sécurité
- Mapper des lecteurs réseau

✅ **Déléguer l'administration**
- Permettre au manager RH de gérer les comptes RH
- Donner au help desk le droit de réinitialiser les mots de passe

✅ **Séparer des environnements**
- OU Production vs OU Test
- OU Serveurs vs OU Workstations

> [!example] Scénario OU
> Vous voulez que tous les ordinateurs du département Marketing aient automatiquement Adobe Creative Suite installé, et que le fond d'écran soit le logo de l'entreprise.
> 
> **Solution** : Créez une OU "Marketing-Computers" et appliquez une GPO avec ces paramètres.

### 👥 Groupes de sécurité : Permissions et accès

**Utilisez un groupe de sécurité pour** :

✅ **Gérer les permissions sur les ressources**
- Accès à un dossier partagé
- Permissions sur une imprimante
- Droits sur une base de données

✅ **Simplifier l'attribution de droits**
- Au lieu de donner des droits à 50 utilisateurs individuellement
- Donnez les droits au groupe, ajoutez les 50 utilisateurs au groupe

✅ **Faciliter la gestion des accès**
- Un employé rejoint le Marketing → ajoutez-le aux groupes Marketing
- Un employé quitte → retirez-le des groupes

✅ **Nesting (imbrication de groupes)**
- Créer des hiérarchies de permissions complexes

> [!example] Scénario groupe
> Vous avez un dossier partagé `\\serveur\Marketing` qui doit être accessible en lecture-écriture uniquement par les employés du Marketing.
> 
> **Solution** : Créez un groupe "GRP-Marketing-RW", donnez les permissions NTFS à ce groupe, ajoutez les employés Marketing comme membres.

### 🔄 OU ET groupes : Complémentaires

Dans la pratique, on utilise **les deux en combinaison** :

**Exemple d'architecture complète** :

```
Structure OU :
OU=Marketing
├── Users
│   ├── Jean.Dupont (membre de GRP-Marketing-Users)
│   ├── Marie.Martin (membre de GRP-Marketing-Users, GRP-Marketing-Managers)
│   └── Pierre.Durand (membre de GRP-Marketing-Users)
└── Computers
    ├── PC-MKT-01
    └── PC-MKT-02

Groupes de sécurité :
- GRP-Marketing-Users (accès lecture au dossier \\serveur\Marketing)
- GRP-Marketing-Managers (accès lecture-écriture + suppression)
- GRP-Marketing-IT (droits d'administration locale sur les PC Marketing)
```

**GPO appliquée à l'OU** :
- Installe Adobe Creative Suite
- Configure le proxy web
- Mappe le lecteur M: vers \\serveur\Marketing

**Permissions via groupes** :
- Dossier \\serveur\Marketing : GRP-Marketing-Users (Read), GRP-Marketing-Managers (Modify)
- Imprimante Couleur Marketing : GRP-Marketing-Users (Print)

### 📋 Tableau de décision

| Question | Utiliser OU | Utiliser Groupe |
|----------|-------------|-----------------|
| Appliquer une GPO ? | ✅ | ❌ |
| Donner accès à un dossier partagé ? | ❌ | ✅ |
| Organiser visuellement l'annuaire ? | ✅ | ❌ |
| Déléguer la gestion de comptes ? | ✅ | ⚠️ |
| Gérer des permissions NTFS ? | ❌ | ✅ |
| Distribuer un logiciel ? | ✅ (via GPO) | ❌ |
| Envoyer un email à tout le département ? | ❌ | ✅ (groupe distribution) |
| Restreindre l'accès à une application ? | ❌ | ✅ |
| Séparer Production et Test ? | ✅ | ⚠️ |

### 🎯 Stratégies combinées

**Stratégie 1 : OU par département + Groupes pour permissions**

```
OU Structure :
└── OU=Utilisateurs
    ├── OU=RH
    ├── OU=IT
    └── OU=Marketing

Groupes :
- GRP-RH-Users (permissions sur \\serveur\RH)
- GRP-IT-Users (permissions sur \\serveur\IT)
- GRP-Marketing-Users (permissions sur \\serveur\Marketing)
- GRP-All-Employees (permissions communes)
```

**Avantages** :
- GPO ciblées par département (via OU)
- Permissions flexibles (via groupes)
- Facile à gérer

**Stratégie 2 : OU par localisation + Groupes par fonction**

```
OU Structure :
└── OU=Sites
    ├── OU=Paris
    │   ├── Users (tous départements)
    │   └── Computers
    └── OU=Londres
        ├── Users (tous départements)
        └── Computers

Groupes :
- GRP-Global-RH (tous les RH, tous sites)
- GRP-Global-IT (tous les IT, tous sites)
- GRP-Paris-LocalAdmin (admins locaux à Paris)
```

**Avantages** :
- GPO par site (fuseaux horaires, imprimantes locales)
- Permissions globales par fonction (accès aux applications)
- Support multi-site simplifié

### 💡 Cas pratiques

**Cas 1 : Nouveau dossier partagé pour le projet "Neptune"**

❌ **Mauvaise approche** : Créer une OU "Projet-Neptune"
- Les OU ne gèrent pas les permissions de fichiers
- Les utilisateurs du projet peuvent être dans différentes OU (RH, IT, Marketing)

✅ **Bonne approche** : Créer un groupe "GRP-Projet-Neptune"
- Ajouter tous les membres du projet au groupe (peu importe leur OU)
- Donner les permissions au groupe
- Facilite l'ajout/retrait de membres

**Cas 2 : Déployer Office 365 aux employés de Londres uniquement**

❌ **Mauvaise approche** : Créer un groupe "Employees-Londres"
- Les groupes ne reçoivent pas les GPO directement

✅ **Bonne approche** : Créer une OU "Londres-Users"
- Déplacer les utilisateurs de Londres dans cette OU
- Créer une GPO de déploiement Office 365
- Lier la GPO à l'OU "Londres-Users"

**Cas 3 : Gérer l'accès à une application web (SharePoint)**

❌ **Mauvaise approche** : Donner les permissions basées sur les OU
- SharePoint ne comprend pas les OU
- Complexe à maintenir

✅ **Bonne approche** : Créer des groupes de sécurité
- GRP-SharePoint-Readers
- GRP-SharePoint-Contributors
- GRP-SharePoint-Admins
- Utiliser ces groupes dans les permissions SharePoint

### 🚫 Erreurs à éviter

**Erreur 1 : Dupliquer la structure en OU et en groupes**

❌ **Mauvais** :
```
OU=Marketing + Groupe=GRP-Marketing-OU
OU=RH + Groupe=GRP-RH-OU
OU=IT + Groupe=GRP-IT-OU
```

Inutile et source de confusion. Les OU et groupes ont des rôles différents.

**Erreur 2 : Utiliser uniquement des groupes, pas d'OU**

❌ **Mauvais** :
```
Tous les utilisateurs dans CN=Users
Gestion uniquement par groupes
```

Impossible d'appliquer des GPO ciblées, pas de délégation structurée.

**Erreur 3 : Créer une OU pour chaque groupe**

❌ **Mauvais** :
```
OU=Projet-A
OU=Projet-B
OU=Projet-C
...
```

Les projets sont temporaires, créent une pollution de l'annuaire. Utilisez des groupes.

### 📊 Récapitulatif visuel

```
┌─────────────────────────────────────────────┐
│           ACTIVE DIRECTORY                  │
│                                             │
│  OU (Organisation)          Groupe (Accès)  │
│  ┌─────────────────┐       ┌──────────────┐│
│  │ OU=Marketing    │       │GRP-Marketing ││
│  │  ├── Users      │       │ Membres:     ││
│  │  ├── Computers  │       │  - Jean      ││
│  │  └── Groupes    │       │  - Marie     ││
│  └─────────────────┘       │  - Pierre    ││
│         ↓                  └──────────────┘│
│      Applique                     ↓        │
│         GPO                   Permissions   │
│         ↓                         ↓        │
│  ┌─────────────────┐       ┌──────────────┐│
│  │ - Install Apps  │       │ \\serveur\   ││
│  │ - Wallpaper     │       │   Marketing  ││
│  │ - Proxy         │       │   (NTFS)     ││
│  └─────────────────┘       └──────────────┘│
└─────────────────────────────────────────────┘
```

---

## 🎯 Récapitulatif

Les Unités d'Organisation sont un élément central de la gestion d'Active Directory :

### ✅ Points clés à retenir

**Concept des OU** :
- Conteneurs pour organiser les objets AD
- Triple fonction : organisation, GPO, délégation
- Hiérarchiques et flexibles
- Différentes des conteneurs par défaut

**Création et structure** :
- Planifier avant de créer
- Limiter à 3-4 niveaux de profondeur
- Standardiser les noms
- Protéger contre la suppression accidentelle

**Délégation de contrôle** :
- Permet l'administration distribuée
- Principe du moindre privilège
- Toujours utiliser des groupes pour déléguer
- Documenter et auditer régulièrement

**Bonnes pratiques** :
- Simplicité et cohérence
- Structure orientée GPO
- Séparer les types d'objets
- Documenter la structure

**OU vs Groupes** :
- OU = Organisation + GPO + Délégation
- Groupes = Permissions + Accès
- Complémentaires, pas interchangeables
- Utiliser les deux en combinaison

> [!tip] Conseil final
> Une bonne structure d'OU est invisible : elle facilite le travail quotidien sans qu'on y pense. Si vous passez votre temps à chercher des objets ou à créer des OU spéciales, votre structure a probablement besoin d'être simplifiée.

---

*Cours créé pour Obsidian - Active Directory - Partie 5*