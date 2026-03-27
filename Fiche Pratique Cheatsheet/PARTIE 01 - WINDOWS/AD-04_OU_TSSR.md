# AD-04 — Unités d'Organisation (OU)

> Les OU sont des **conteneurs** qui structurent l'annuaire AD logiquement.  
> Elles servent à 3 choses : **organiser**, **appliquer des GPO**, **déléguer l'administration**.

---

## Définition & rôle

| Rôle | Description |
|------|-------------|
| **Organisation logique** | Refléter la structure de l'entreprise (services, sites...) |
| **Application de GPO** | Lier des stratégies de groupe à un périmètre précis |
| **Délégation d'administration** | Confier la gestion d'une OU à un utilisateur ou groupe |

> Différence clé : une OU **n'est pas** un groupe — elle ne sert pas aux permissions NTFS.

### Ce qu'une OU peut contenir

- Utilisateurs
- Ordinateurs
- Groupes
- Imprimantes
- Autres OU (imbrication possible)

---

## Modèles d'organisation

### Par service (le plus courant)

```
lab.lan
+-- OU=LabUsers
|   +-- OU=DirectionCommerciale
|   +-- OU=DirectionRH
|   +-- OU=DirectionIT
+-- OU=LabComputers
|   +-- OU=PC_Bureau
|   +-- OU=PC_Portable
|   +-- OU=Serveurs
+-- OU=Groupes
+-- OU=DeactivatedUsers
```

### Par localisation géographique

```
lab.lan
+-- OU=Paris
|   +-- OU=Utilisateurs
|   +-- OU=Ordinateurs
+-- OU=Lyon
|   +-- OU=Utilisateurs
|   +-- OU=Ordinateurs
+-- OU=Nantes
```

### Hybride (service + localisation)

```
lab.lan
+-- OU=France
|   +-- OU=Paris
|   |   +-- OU=RH
|   |   +-- OU=IT
|   +-- OU=Lyon
+-- OU=Belgique
```

---

## GUI — Gestion des OU via ADUC

### Ouvrir ADUC

```
Win + R -> dsa.msc
# ou
Gestionnaire de serveur -> Outils -> Utilisateurs et ordinateurs Active Directory
```

### Créer une OU (GUI)

1. Clic droit sur le domaine ou une OU parente
2. `Nouveau` -> `Unité d'organisation`
3. Saisir le **nom** de l'OU
4. Option : cocher `Protéger le conteneur contre une suppression accidentelle` (✅ recommandé)
5. `OK`

### Créer une sous-OU (GUI)

1. Clic droit sur l'**OU parente** existante
2. `Nouveau` -> `Unité d'organisation`
3. Saisir le nom -> `OK`

### Renommer une OU (GUI)

1. Clic droit sur l'OU -> `Renommer`
2. Saisir le nouveau nom -> `Entrée`

### Déplacer une OU (GUI)

1. Clic droit sur l'OU -> `Déplacer`
2. Sélectionner la destination -> `OK`

### Supprimer une OU (GUI)

> ⚠️ Si la protection est activée, il faut d'abord la désactiver.

1. Activer `Affichage` -> `Fonctionnalités avancées` dans ADUC
2. Double-clic sur l'OU -> onglet `Objet`
3. Décocher `Protéger l'objet contre une suppression accidentelle`
4. `OK`
5. Clic droit sur l'OU -> `Supprimer` -> Confirmer

### Déplacer un objet dans une OU (GUI)

1. Clic droit sur l'objet (utilisateur, ordinateur...) -> `Déplacer`
2. Sélectionner l'OU de destination -> `OK`

---

## GUI — Délégation de contrôle

> Permet de confier des tâches d'administration à un utilisateur ou groupe **sans lui donner les droits Domain Admins**.

### Procédure : Déléguer le contrôle d'une OU (GUI)

1. Clic droit sur l'**OU cible** -> `Délégation de contrôle`
2. Assistant de délégation -> `Suivant`
3. `Ajouter` -> Rechercher l'utilisateur ou le groupe -> `OK` -> `Suivant`
4. Choisir les tâches à déléguer :

| Tâche prédéfinie | Usage typique |
|-----------------|---------------|
| Créer, supprimer et gérer des comptes d'utilisateurs | Helpdesk RH |
| Réinitialiser les mots de passe | Helpdesk niveau 1 |
| Lire toutes les informations des utilisateurs | Support lecture seule |
| Créer, supprimer et gérer des groupes | Admin groupes |
| Gérer l'appartenance aux groupes | Responsable service |
| Tâche personnalisée (mode avancé) | Droits fins sur mesure |

5. `Suivant` -> `Terminer`

---

## PowerShell — Gestion des OU

```powershell
# Créer une OU
New-ADOrganizationalUnit -Name "DirectionIT" `
                         -Path "DC=lab,DC=lan" `
                         -ProtectedFromAccidentalDeletion $true

# Créer une sous-OU
New-ADOrganizationalUnit -Name "Utilisateurs" `
                         -Path "OU=DirectionIT,DC=lab,DC=lan" `
                         -ProtectedFromAccidentalDeletion $true

# Lister toutes les OU
Get-ADOrganizationalUnit -Filter *

# Chercher une OU par nom
Get-ADOrganizationalUnit -Filter {Name -like "Direction*"}

# Afficher le DN complet d'une OU
Get-ADOrganizationalUnit -Filter {Name -eq "DirectionIT"} |
    Select Name, DistinguishedName

# Renommer une OU
Rename-ADObject -Identity "OU=DirectionIT,DC=lab,DC=lan" -NewName "IT"

# Déplacer une OU
Move-ADObject -Identity "OU=DirectionIT,DC=lab,DC=lan" `
              -TargetPath "OU=France,DC=lab,DC=lan"

# Désactiver la protection avant suppression
Set-ADOrganizationalUnit -Identity "OU=DirectionIT,DC=lab,DC=lan" `
                         -ProtectedFromAccidentalDeletion $false

# Supprimer une OU (protection désactivée au préalable)
Remove-ADOrganizationalUnit -Identity "OU=DirectionIT,DC=lab,DC=lan" -Confirm:$false

# Déplacer un utilisateur dans une OU
Move-ADObject -Identity "CN=Jean Dupont,OU=Compta,DC=lab,DC=lan" `
              -TargetPath "OU=DirectionIT,DC=lab,DC=lan"

# Lister tous les utilisateurs d'une OU
Get-ADUser -Filter * -SearchBase "OU=DirectionIT,DC=lab,DC=lan"

# Lister tous les objets d'une OU (récursif)
Get-ADObject -Filter * -SearchBase "OU=DirectionIT,DC=lab,DC=lan" `
             -SearchScope Subtree | Select Name, ObjectClass
```

---

## Différence OU vs Groupe vs Conteneur

| Critère | OU | Groupe | Conteneur (CN) |
|---------|-----|--------|----------------|
| **Objectif** | Organisation + GPO + délégation | Permissions NTFS / email | Organisation native AD |
| **Application GPO** | ✅ Oui | ❌ Non | ❌ Non |
| **Délégation** | ✅ Oui | ❌ Non | ❌ Non |
| **Permissions NTFS** | ❌ Non | ✅ Oui | ❌ Non |
| **Imbrication** | ✅ Oui | ✅ Oui | Limitée |
| **Créé par admin** | ✅ Oui | ✅ Oui | ❌ Natif AD uniquement |

> Les **Containers** (comme `CN=Users`, `CN=Computers`) sont créés automatiquement par AD et **ne peuvent pas recevoir de GPO**.

---

## Bonnes pratiques

| Pratique | Raison |
|----------|--------|
| Toujours activer la **protection contre suppression accidentelle** | Évite les destructions involontaires |
| Créer une OU **DeactivatedUsers** | Stocker les comptes désactivés |
| Séparer OU **Utilisateurs** / **Ordinateurs** / **Groupes** | Facilite l'application des GPO |
| Nommer les OU de façon **explicite et cohérente** | Lisibilité et maintenance |
| Réfléchir à la structure **avant** de déployer | Difficile à réorganiser en production |
| Ne pas utiliser les conteneurs natifs `CN=Users` / `CN=Computers` | Pas de GPO applicable |

---

## ⚠️ À retenir absolument

- `dsa.msc` = ouvrir ADUC (gestion des OU en GUI)
- Une OU **n'est pas un groupe** — pas de permissions NTFS via une OU
- Les **Containers** (`CN=Users`, `CN=Computers`) ne supportent **pas** les GPO → toujours utiliser des OU
- `ProtectedFromAccidentalDeletion $true` = protection obligatoire en production
- Pour supprimer une OU protégée : d'abord désactiver la protection via `Affichage` -> `Fonctionnalités avancées`
- `-SearchBase` dans PowerShell = limiter la recherche à une OU précise
- La délégation de contrôle = principe de **moindre privilège** appliqué aux OU
- Structure OU = doit refléter ce qui aide à **appliquer les GPO**, pas forcément l'organigramme exact
