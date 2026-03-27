

> [!info] Vue d'ensemble
> Les stratégies de groupe (Group Policy Objects - GPO) constituent le mécanisme central de gestion et de configuration des utilisateurs et ordinateurs dans Active Directory. Cette section couvre les concepts fondamentaux, la structure, la liaison, l'héritage et les outils de gestion des GPO.

---

## 📑 Table des matières

- [[#🎯 Concept des GPO|Concept des GPO]]
- [[#🏗️ Structure d'une GPO|Structure d'une GPO]]
- [[#🔗 Liaison de GPO (Link) aux OU|Liaison de GPO (Link) aux OU]]
- [[#📊 Héritage et Ordre de Traitement|Héritage et Ordre de Traitement]]
- [[#🛡️ Forcer et Bloquer l'Héritage|Forcer et Bloquer l'Héritage]]
- [[#🔒 Filtrage de Sécurité|Filtrage de Sécurité]]
- [[#🛠️ Outils de Gestion (GPMC)|Outils de Gestion (GPMC)]]

---

## 🎯 Concept des GPO

Les **Group Policy Objects (GPO)** sont des collections de paramètres qui définissent ce que les utilisateurs peuvent faire et comment les systèmes sont configurés dans un environnement Active Directory.

### Qu'est-ce qu'une GPO ?

> [!info] Définition
> Une GPO est un ensemble de règles et de paramètres qui permettent de :
> - **Configurer** des paramètres système et utilisateur
> - **Restreindre** ou **autoriser** des actions
> - **Déployer** des logiciels
> - **Appliquer** des politiques de sécurité
> - **Automatiser** des tâches administratives
> 
> Les GPO s'appliquent automatiquement aux utilisateurs et ordinateurs en fonction de leur emplacement dans Active Directory.

### Pourquoi Utiliser les GPO ?

**Avantages principaux :**

| Avantage | Description | Exemple |
|----------|-------------|---------|
| **Centralisation** | Gestion depuis un point unique | Configuration fond d'écran pour 1000 postes |
| **Cohérence** | Paramètres identiques partout | Politique mot de passe uniforme |
| **Automatisation** | Application automatique | Installation logiciels au démarrage |
| **Sécurité** | Renforcement standardisé | Blocage USB sur postes sensibles |
| **Efficacité** | Économie de temps et efforts | Configuration en masse vs manuelle |
| **Conformité** | Respect des standards | Application normes ISO/PCI-DSS |

> [!example] Cas d'Usage Concrets
> **Scénarios courants :**
> 
> 1. **Sécurité :**
>    - Exiger des mots de passe complexes
>    - Verrouiller les sessions après 10 minutes d'inactivité
>    - Désactiver l'USB sur certains postes
> 
> 2. **Configuration :**
>    - Définir un fond d'écran d'entreprise
>    - Mapper des lecteurs réseau automatiquement
>    - Configurer les imprimantes par défaut
> 
> 3. **Déploiement :**
>    - Installer Office automatiquement
>    - Déployer des mises à jour
>    - Distribuer des scripts de connexion
> 
> 4. **Restrictions :**
>    - Bloquer l'accès au panneau de configuration
>    - Empêcher l'installation de logiciels
>    - Masquer certaines options du menu démarrer

### Types de Paramètres GPO

Les GPO contiennent deux grandes catégories de paramètres :

#### 1. Configuration Ordinateur (Computer Configuration)

> [!info] Configuration Ordinateur
> **S'applique** : À l'ordinateur, indépendamment de qui se connecte
> 
> **Moment d'application** : Au démarrage de l'ordinateur
> 
> **Exemples :**
> - Paramètres de sécurité du système
> - Installation de logiciels (pour tous les utilisateurs)
> - Scripts de démarrage
> - Configuration du pare-feu
> - Paramètres réseau
> - Services Windows

```
Computer Configuration
├── Policies
│   ├── Software Settings
│   ├── Windows Settings
│   │   ├── Security Settings
│   │   ├── Scripts (Startup/Shutdown)
│   │   └── Policy-based QoS
│   └── Administrative Templates
│       ├── Control Panel
│       ├── Network
│       ├── System
│       └── Windows Components
└── Preferences
    ├── Control Panel Settings
    ├── Windows Settings
    └── ...
```

#### 2. Configuration Utilisateur (User Configuration)

> [!info] Configuration Utilisateur
> **S'applique** : À l'utilisateur qui se connecte
> 
> **Moment d'application** : À l'ouverture de session
> 
> **Exemples :**
> - Redirection de dossiers (Documents, Bureau)
> - Lecteurs réseau mappés
> - Paramètres Internet Explorer/Edge
> - Fond d'écran et thème
> - Scripts de connexion/déconnexion
> - Restrictions sur le bureau

```
User Configuration
├── Policies
│   ├── Software Settings
│   ├── Windows Settings
│   │   ├── Scripts (Logon/Logoff)
│   │   ├── Security Settings
│   │   ├── Folder Redirection
│   │   └── Policy-based QoS
│   └── Administrative Templates
│       ├── Control Panel
│       ├── Desktop
│       ├── Network
│       ├── Start Menu and Taskbar
│       └── Windows Components
└── Preferences
    ├── Control Panel Settings
    ├── Windows Settings
    └── ...
```

> [!tip] Quelle Configuration Choisir ?
> **Computer Configuration** quand :
> - Le paramètre doit s'appliquer quel que soit l'utilisateur
> - Configuration système ou matérielle
> - Sécurité globale de la machine
> 
> **User Configuration** quand :
> - Le paramètre est spécifique à un utilisateur
> - Personnalisation de l'expérience utilisateur
> - Droits ou restrictions par profil

### GPO par Défaut

Active Directory crée automatiquement deux GPO lors de la création du domaine :

#### Default Domain Policy

> [!info] Default Domain Policy
> **Portée** : Liée à la racine du domaine
> 
> **Objectif** : Définir les politiques de sécurité du domaine
> 
> **Paramètres typiques :**
> - Politique de mot de passe (complexité, âge, historique)
> - Politique de verrouillage de compte
> - Politique Kerberos
> - Droits utilisateurs au niveau domaine
> 
> **⚠️ Bonne pratique :** Ne modifier que pour paramètres de sécurité du domaine

```powershell
# Voir les paramètres de Default Domain Policy
Get-GPO -Name "Default Domain Policy"

# Générer un rapport
Get-GPOReport -Name "Default Domain Policy" -ReportType Html -Path "C:\Reports\DefaultDomainPolicy.html"
```

#### Default Domain Controllers Policy

> [!info] Default Domain Controllers Policy
> **Portée** : Liée à l'OU "Domain Controllers"
> 
> **Objectif** : Paramètres spécifiques aux contrôleurs de domaine
> 
> **Paramètres typiques :**
> - Droits utilisateurs pour les DC
> - Audit et journalisation
> - Restrictions de sécurité
> 
> **⚠️ Bonne pratique :** Ne modifier que pour paramètres spécifiques aux DC

> [!warning] Ne Pas Modifier sans Raison
> Ces GPO par défaut ont une configuration éprouvée. Les modifier peut :
> - Compromettre la sécurité du domaine
> - Causer des dysfonctionnements
> - Compliquer le dépannage
> 
> **Recommandation :** Créer de nouvelles GPO plutôt que modifier les GPO par défaut.

### Domaines d'Application GPO

Les GPO peuvent être liées à différents niveaux de la structure AD :

```
Forêt Active Directory
│
└── Domaine (entreprise.local)
    ├── [GPO liée au domaine] ← Tous les objets du domaine
    │
    ├── OU=Paris
    │   ├── [GPO liée à Paris] ← Tous les objets sous Paris
    │   │
    │   ├── OU=Services
    │   │   ├── [GPO liée à Services] ← Tous les objets sous Services
    │   │   │
    │   │   ├── OU=IT
    │   │   │   └── [GPO liée à IT] ← Uniquement objets IT
    │   │   │
    │   │   └── OU=RH
    │   │       └── [GPO liée à RH] ← Uniquement objets RH
    │   │
    │   └── OU=Workstations
    │       └── [GPO liée à Workstations]
    │
    └── OU=Lyon
        └── [GPO liée à Lyon]
```

> [!info] Points de Liaison Possibles
> - **Site** : Tous les objets d'un site physique (rare)
> - **Domaine** : Tous les objets du domaine
> - **OU** : Objets contenus dans l'OU (le plus courant)
> 
> ❌ **Impossible** : Lier GPO directement à un utilisateur ou ordinateur individuel

---

## 🏗️ Structure d'une GPO

Une GPO est composée de deux parties distinctes mais complémentaires qui travaillent ensemble.

### Architecture d'une GPO

> [!info] Composants d'une GPO
> Chaque GPO se compose de :
> 
> 1. **GPC (Group Policy Container)** : Objet dans Active Directory
> 2. **GPT (Group Policy Template)** : Fichiers dans SYSVOL
> 
> Ces deux parties sont liées par un GUID (identifiant unique global).

```
GPO "Sécurité Postes IT"
│
├── GPC (Active Directory)
│   ├── Nom: Sécurité Postes IT
│   ├── GUID: {12345678-1234-1234-1234-123456789ABC}
│   ├── Version
│   ├── Statut (Enabled/Disabled)
│   └── Liens (OU où la GPO est appliquée)
│
└── GPT (SYSVOL)
    └── \\entreprise.local\SYSVOL\entreprise.local\Policies\{GUID}\
        ├── GPT.INI (version)
        ├── Machine\ (Configuration Ordinateur)
        │   ├── Registry.pol
        │   ├── Scripts\
        │   └── ...
        └── User\ (Configuration Utilisateur)
            ├── Registry.pol
            ├── Scripts\
            └── ...
```

### 1. GPC (Group Policy Container)

Le **GPC** est stocké dans Active Directory sous forme d'objet.

```powershell
# Voir tous les GPC dans AD
Get-ADObject -Filter {objectClass -eq "groupPolicyContainer"} -Properties DisplayName, gPCFileSysPath

# Voir le GPC d'une GPO spécifique
Get-ADObject -Filter {DisplayName -eq "Sécurité Postes IT"} -Properties *
```

**Emplacement dans AD :**
```
CN=Policies,CN=System,DC=entreprise,DC=local
├── CN={GUID-GPO-1}
├── CN={GUID-GPO-2}
└── CN={GUID-GPO-3}
```

**Informations stockées dans le GPC :**
- Nom de la GPO (DisplayName)
- GUID unique
- Numéro de version
- État (activé/désactivé)
- Informations de liaison
- Permissions
- Filtres WMI

### 2. GPT (Group Policy Template)

Le **GPT** contient les fichiers de configuration réels de la GPO.

**Emplacement :**
```
\\domaine.local\SYSVOL\domaine.local\Policies\{GUID}\
```

```powershell
# Accéder au GPT
$gpoGUID = "{12345678-1234-1234-1234-123456789ABC}"
$gptPath = "\\entreprise.local\SYSVOL\entreprise.local\Policies\$gpoGUID"
explorer $gptPath
```

**Structure du GPT :**

```
{GUID}\
├── GPT.INI                     # Fichier de version
├── Machine\                    # Configuration Ordinateur
│   ├── comment.cmtx
│   ├── Registry.pol           # Paramètres registre ordinateur
│   ├── Microsoft\
│   │   └── Windows NT\
│   │       └── SecEdit\
│   │           └── GptTmpl.inf # Modèles de sécurité
│   ├── Scripts\
│   │   ├── Startup\           # Scripts de démarrage
│   │   └── Shutdown\          # Scripts d'arrêt
│   └── Preferences\           # Préférences ordinateur
│
├── User\                      # Configuration Utilisateur
│   ├── Registry.pol           # Paramètres registre utilisateur
│   ├── Scripts\
│   │   ├── Logon\             # Scripts de connexion
│   │   └── Logoff\            # Scripts de déconnexion
│   └── Preferences\           # Préférences utilisateur
│
└── Backup.xml                 # Métadonnées de sauvegarde
```

#### Fichier GPT.INI

```ini
[General]
Version=196613
displayName=Sécurité Postes IT
```

**Décryptage du numéro de version :**
- Format : `ADM_VERSION * 65536 + USR_VERSION`
- `196613` = `3 * 65536 + 5`
  - Version Computer : 3
  - Version User : 5

> [!info] Synchronisation GPC/GPT
> Le numéro de version **doit être identique** dans le GPC (AD) et le GPT (SYSVOL).
> 
> Si les versions diffèrent :
> - Problème de réplication
> - GPO potentiellement corrompue
> - Paramètres pourraient ne pas s'appliquer correctement

### Fichiers Registry.pol

Ces fichiers binaires contiennent les paramètres de registre à appliquer.

```powershell
# Voir le contenu de Registry.pol (nécessite Parse-PolFile ou LGPO)
# Exemple avec module PolicyFileEditor
Install-Module -Name PolicyFileEditor

# Lire les paramètres
Get-PolicyFileEntry -Path "\\entreprise.local\SYSVOL\entreprise.local\Policies\{GUID}\Machine\Registry.pol"
```

> [!warning] Ne Pas Modifier Manuellement
> Les fichiers `.pol` sont au format binaire propriétaire.
> - ❌ Ne jamais éditer directement
> - ✅ Utiliser la console GPMC ou PowerShell
> - Modification directe = corruption quasi-certaine

### GUID de la GPO

Chaque GPO possède un identifiant unique (GUID) qui lie le GPC et le GPT.

```powershell
# Voir le GUID d'une GPO
Get-GPO -Name "Sécurité Postes IT" | Select-Object DisplayName, Id

# Résultat exemple
DisplayName         Id
-----------         --
Sécurité Postes IT  12345678-1234-1234-1234-123456789abc

# Trouver une GPO par son GUID
Get-GPO -Guid "12345678-1234-1234-1234-123456789abc"
```

### Réplication GPO

> [!info] Réplication sur Plusieurs DC
> **GPC :**
> - Répliqué via réplication Active Directory normale
> - Latence : Selon topologie de réplication AD
> 
> **GPT :**
> - Répliqué via DFS-R (ou anciennement FRS)
> - SYSVOL partagé entre tous les DC
> - Latence : Variable selon bande passante
> 
> **Problème potentiel :**
> GPC et GPT peuvent être désynchronisés temporairement entre DC.

```powershell
# Vérifier la réplication SYSVOL
dfsrdiag ReplicationState

# Forcer la réplication SYSVOL
repadmin /syncall /AdeP

# Vérifier l'état de réplication AD
repadmin /replsummary
```

### Taille et Limites GPO

> [!warning] Limites Techniques
> **Taille Registry.pol :**
> - Limite théorique : ~100 MB
> - Recommandé : < 1 MB
> - Au-delà : performances dégradées
> 
> **Nombre de paramètres :**
> - Pas de limite stricte
> - Recommandé : < 1000 paramètres par GPO
> 
> **Nombre de GPO par domaine :**
> - Limite pratique : ~1000-2000 GPO
> - Au-delà : difficultés de gestion

---

## 🔗 Liaison de GPO (Link) aux OU

La **liaison** (link) est l'action d'associer une GPO à un conteneur AD (domaine, OU) pour qu'elle s'applique aux objets qu'il contient.

### Concept de Liaison

> [!info] Qu'est-ce qu'une Liaison ?
> Une **liaison de GPO** est un pointeur depuis un conteneur AD (OU, domaine, site) vers une GPO.
> 
> **Caractéristiques :**
> - Une GPO peut être liée à plusieurs OU
> - Une OU peut avoir plusieurs GPO liées
> - Les GPO existent indépendamment des liaisons
> - Supprimer une liaison ne supprime pas la GPO

```
GPO "Sécurité Standard"
    ↓ (lien 1)
    OU=IT
    ↓ (lien 2)
    OU=RH
    ↓ (lien 3)
    OU=Compta

Une seule GPO, trois liens différents
```

### Lier une GPO via GPMC

**Étapes dans la console GPMC :**

```
1. Ouvrir Group Policy Management (gpmc.msc)
2. Développer la forêt → Domains → votre domaine
3. Naviguer vers l'OU cible
4. Clic droit sur l'OU → "Link an Existing GPO..."
5. Sélectionner la GPO dans la liste
6. Cliquer OK
```

> [!tip] Raccourci
> Vous pouvez aussi glisser-déposer une GPO depuis le conteneur "Group Policy Objects" vers une OU.

### Lier une GPO via PowerShell

```powershell
# Lier une GPO à une OU
New-GPLink -Name "Sécurité Postes IT" `
           -Target "OU=IT,OU=Services,DC=entreprise,DC=local"

# Lier au domaine
New-GPLink -Name "Politique Mot de Passe" `
           -Target "DC=entreprise,DC=local"

# Lier avec ordre spécifique (LinkOrder, voir section héritage)
New-GPLink -Name "Configuration Bureau" `
           -Target "OU=Workstations,DC=entreprise,DC=local" `
           -LinkEnabled Yes `
           -Order 1

# Lier et forcer immédiatement
New-GPLink -Name "Sécurité Critique" `
           -Target "OU=Serveurs,DC=entreprise,DC=local" `
           -Enforced Yes
```

### Activer/Désactiver une Liaison

```powershell
# Désactiver un lien (GPO ne s'applique plus, mais lien reste)
Set-GPLink -Name "Ancienne Config" `
           -Target "OU=IT,DC=entreprise,DC=local" `
           -LinkEnabled No

# Réactiver un lien
Set-GPLink -Name "Ancienne Config" `
           -Target "OU=IT,DC=entreprise,DC=local" `
           -LinkEnabled Yes

# Voir l'état d'un lien
Get-GPLink -Target "OU=IT,DC=entreprise,DC=local"
```

> [!info] Désactiver vs Supprimer
> **Désactiver un lien :**
> - Le lien existe toujours
> - Peut être réactivé facilement
> - Utile pour tests ou maintenance
> 
> **Supprimer un lien :**
> - Le lien disparaît complètement
> - GPO n'apparaît plus dans l'OU
> - Peut être recréé mais perd son ordre

### Supprimer une Liaison

```powershell
# Supprimer un lien GPO
Remove-GPLink -Name "Config Obsolète" `
              -Target "OU=IT,DC=entreprise,DC=local"

# Via GPMC : Clic droit sur GPO dans l'OU → Delete (supprime le lien, pas la GPO)
```

> [!warning] Supprimer Lien vs Supprimer GPO
> **Dans GPMC, depuis l'OU :**
> - Clic droit → Delete = Supprime uniquement le **lien**
> 
> **Dans GPMC, depuis "Group Policy Objects" :**
> - Clic droit → Delete = Supprime la **GPO entière** (tous ses liens aussi)

### Vérifier les Liaisons

```powershell
# Lister toutes les GPO liées à une OU
Get-GPInheritance -Target "OU=IT,DC=entreprise,DC=local"

# Lister tous les liens d'une GPO spécifique
(Get-GPO -Name "Sécurité Postes IT").GetGPOLinks()

# Alternative plus détaillée
$gpo = Get-GPO -Name "Sécurité Postes IT"
Get-ADOrganizationalUnit -Filter * | 
    Where-Object {(Get-GPInheritance -Target $_.DistinguishedName).GpoLinks.DisplayName -contains $gpo.DisplayName} |
    Select-Object Name, DistinguishedName

# Rapport complet de toutes les liaisons du domaine
Get-GPO -All | ForEach-Object {
    $gpo = $_
    $links = $gpo | Get-GPOReport -ReportType Xml | Select-Xml -XPath "//LinksTo"
    if ($links) {
        [PSCustomObject]@{
            GPOName = $gpo.DisplayName
            Links = ($links | ForEach-Object {$_.Node.SOMPath}) -join "; "
        }
    }
} | Format-Table -AutoSize
```

### Ordre de Liaison (Link Order)

Quand plusieurs GPO sont liées à la même OU, l'**ordre de liaison** détermine leur priorité.

> [!info] Ordre de Liaison
> **Principe :**
> - Ordre 1 = Priorité la plus haute
> - Ordre 2 = Priorité inférieure
> - Ordre 3 = Encore moins prioritaire
> 
> **En cas de conflit :**
> - L'ordre 1 gagne sur ordre 2
> - L'ordre 2 gagne sur ordre 3

```
OU=IT (3 GPO liées)
├── [1] GPO-IT-Sécurité-Critique   ← Plus haute priorité
├── [2] GPO-IT-Configuration
└── [3] GPO-IT-Logiciels           ← Plus basse priorité
```

```powershell
# Modifier l'ordre de liaison
Set-GPLink -Name "GPO-IT-Sécurité-Critique" `
           -Target "OU=IT,DC=entreprise,DC=local" `
           -Order 1

# Déplacer au premier rang (top)
Set-GPLink -Name "Nouvelle GPO Importante" `
           -Target "OU=IT,DC=entreprise,DC=local" `
           -Order 1

# Les autres GPO sont automatiquement décalées vers le bas
```

### GPO Non Liées

> [!tip] GPO Sans Lien
> Il est possible (et courant) d'avoir des GPO créées mais pas encore liées.
> 
> **Utilisations :**
> - GPO en cours de développement/test
> - Templates GPO pour clonage
> - GPO sauvegardées pour restauration future
> - GPO désactivées temporairement
> 
> **Voir les GPO non liées :**

```powershell
# Lister les GPO sans aucun lien
Get-GPO -All | Where-Object {
    $gpo = $_
    $links = ($gpo | Get-GPOReport -ReportType Xml | Select-Xml -XPath "//LinksTo").Count
    $links -eq 0
} | Select-Object DisplayName, CreationTime, ModificationTime
```

---

## 📊 Héritage et Ordre de Traitement

L'**héritage** détermine comment les GPO s'appliquent en cascade depuis les niveaux supérieurs vers les niveaux inférieurs de la structure AD.

### Principe de l'Héritage

> [!info] Héritage GPO
> Les GPO appliquées à un niveau parent sont **héritées** par tous les objets enfants, sauf configuration contraire.
> 
> **Flux d'héritage :**
> ```
> Site
>   ↓ (héritage)
> Domaine
>   ↓ (héritage)
> OU Parent
>   ↓ (héritage)
> OU Enfant
>   ↓ (héritage)
> OU Petite-Enfant
> ```

**Exemple concret :**

```
Domaine (entreprise.local)
├── GPO: Politique Mot de Passe ← S'applique à TOUT le domaine
│
└── OU=Paris
    ├── GPO: Config Paris ← S'applique à Paris et sous-OU
    │
    ├── OU=Services
    │   ├── GPO: Config Services ← S'applique à Services et sous-OU
    │   │
    │   └── OU=IT
    │       └── GPO: Config IT ← S'applique uniquement à IT
    │
    └── OU=Workstations

Un ordinateur dans OU=IT reçoit :
✅ Politique Mot de Passe (domaine)
✅ Config Paris (OU parent)
✅ Config Services (OU parent)
✅ Config IT (OU directe)
```

### Ordre de Traitement : LSDOU

> [!important] Acronyme LSDOU
> Les GPO sont traitées dans cet ordre précis :
> 
> **L**ocal → **S**ite → **D**omain → **OU**
> 
> 1. **L**ocal : GPO locale de la machine (si elle existe)
> 2. **S**ite : GPO liées au site AD
> 3. **D**omain : GPO liées au domaine
> 4. **OU** : GPO liées aux OU, du parent vers l'enfant

```
Ordre d'application (du premier au dernier) :

[1] Local Group Policy (gpedit.msc sur la machine locale)
      ↓
[2] Site GPO (ex: Site-Paris)
      ↓
[3] Domain GPO (ex: Default Domain Policy)
      ↓
[4] OU Parent GPO (ex: GPO-Services)
      ↓
[5] OU Enfant GPO (ex: GPO-IT)
      ↓
[6] OU Petite-Enfant GPO (ex: GPO-IT-Admins)

⚠️ Les dernières appliquées (OU enfant) ont priorité en cas de conflit
```

> [!tip] Règle de Priorité
> **La dernière GPO traitée gagne !**
> 
> Si une GPO au niveau domaine configure le fond d'écran en bleu, et une GPO au niveau OU le configure en rouge, l'utilisateur aura un fond rouge (OU appliquée après domaine).

### Exemple Détaillé d'Héritage

```
Domaine: entreprise.local
│
├── GPO-Domaine: "Politique Sécurité"
│   - Complexité mot de passe: Activée
│   - Verrouillage après 5 tentatives
│   - Historique: 24 mots de passe
│
└── OU=Services
    │
    ├── GPO-Services: "Config Services"
    │   - Fond d'écran: logo-services.jpg
    │   - Lecteur Z: \\serveur\partage-services
    │
    └── OU=IT
        │
        └── GPO-IT: "Config IT"
            - Fond d'écran: logo-it.jpg  ← Conflit avec GPO-Services !
            - Lecteur Y: \\serveur\partage-it
            - Accès panneau de configuration: Autorisé

Résultat pour utilisateur dans OU=IT :
✅ Complexité mot de passe: Activée (GPO-Domaine)
✅ Verrouillage: 5 tentatives (GPO-Domaine)
✅ Historique: 24 (GPO-Domaine)
✅ Fond d'écran: logo-it.jpg (GPO-IT gagne sur GPO-Services)
✅ Lecteur Z: Mappé (GPO-Services)
✅ Lecteur Y: Mappé (GPO-IT)
✅ Panneau de configuration: Autorisé (GPO-IT)
```

### Visualiser l'Héritage

```powershell
# Voir toutes les GPO héritées par une OU
Get-GPInheritance -Target "OU=IT,OU=Services,DC=entreprise,DC=local"

# Résultat exemple montre :
# - GPO liées directement à l'OU
# - GPO héritées des OU parentes
# - GPO liées au domaine
# - Ordre de traitement
# - Statut de blocage d'héritage

# Résultat simplifié
Get-GPInheritance -Target "OU=IT,OU=Services,DC=entreprise,DC=local" | 
    Select-Object -ExpandProperty GpoLinks | 
    Select-Object DisplayName, Enabled, Enforced, Order
```

**Dans GPMC (Interface Graphique) :**

```
1. Ouvrir GPMC
2. Sélectionner l'OU
3. Onglet "Group Policy Inheritance"
4. Visualiser :
   - Ordre de précédence
   - GPO avec "Enforced"
   - Blocage d'héritage
   - Liens désactivés
```

### Conflits de Paramètres

> [!warning] Résolution des Conflits
> **Quand deux GPO configurent le même paramètre :**
> 
> **Règle générale :** La dernière GPO appliquée (selon LSDOU) gagne
> 
> **Exception 1 :** GPO avec "Enforced" (voir section suivante)
> **Exception 2 :** "Loopback Processing" activé
> 
> **Paramètres non configurés :**
> - "Not Configured" n'écrase pas une valeur existante
> - "Enabled" ou "Disabled" écrase toujours

**Exemple de conflit :**

| GPO | Niveau | Paramètre: Accès USB | Résultat |
|-----|--------|---------------------|----------|
| GPO-Domaine | Domaine | Disabled (Bloqué) | ❌ Écrasé |
| GPO-IT | OU | Enabled (Autorisé) | ✅ **Appliqué** (dernier) |

**Résultat :** Les utilisateurs IT peuvent utiliser USB malgré GPO domaine.

**Avec Enforced :**

| GPO | Niveau | Paramètre: Accès USB | Enforced | Résultat |
|-----|--------|---------------------|----------|----------|
| GPO-Domaine | Domaine | Disabled (Bloqué) | ✅ Yes | ✅ **Appliqué** (forcé) |
| GPO-IT | OU | Enabled (Autorisé) | No | ❌ Ignoré |

### Configuration Ordinateur vs Utilisateur

> [!info] Traitement Séparé
> **Computer Configuration** et **User Configuration** sont traités **indépendamment**.
> 
> **Computer Configuration :**
> - Appliquée selon l'emplacement de l'**ordinateur** dans AD
> - Au démarrage de la machine
> 
> **User Configuration :**
> - Appliquée selon l'emplacement de l'**utilisateur** dans AD
> - À la connexion de l'utilisateur
> 
> **Scénario :**
> - Ordinateur dans OU=Workstations-IT
> - Utilisateur dans OU=Users-RH
> 
> **Résultat :**
> - Computer Config: GPO liées à Workstations-IT
> - User Config: GPO liées à Users-RH

```powershell
# Voir les GPO appliquées à un ordinateur
gpresult /r /scope:computer

# Voir les GPO appliquées à un utilisateur
gpresult /r /scope:user

# Rapport complet HTML
gpresult /h C:\Temp\GPResult.html
```

---

## 🛡️ Forcer et Bloquer l'Héritage

Active Directory offre deux mécanismes pour contrôler l'héritage des GPO : **Enforced** (forcer) et **Block Inheritance** (bloquer).

### Enforced (Forcer l'Héritage)

> [!info] Enforced (No Override)
> **Objectif :** Garantir qu'une GPO s'applique **même si des GPO enfants tentent de la contredire**.
> 
> **Utilisation :** Paramètres de sécurité critiques qui ne doivent jamais être contournés
> 
> **Effet :** La GPO "Enforced" est appliquée en priorité absolue
> 
> **Symbole GPMC :** Icône de cadenas bleu 🔒

#### Quand Utiliser Enforced ?

**Scénarios typiques :**

| Scénario | GPO | Niveau | Enforced |
|----------|-----|--------|----------|
| Politique mot de passe entreprise | Sécurité Globale | Domaine | ✅ Yes |
| Antivirus obligatoire | Protection Standard | Domaine | ✅ Yes |
| Blocage USB services sensibles | No-USB | OU Finance | ✅ Yes |
| Audit de sécurité obligatoire | Audit Events | Domaine | ✅ Yes |

> [!warning] Utiliser avec Parcimonie
> **Enforced** peut créer des situations difficiles à gérer :
> - Impossible de faire des exceptions dans les OU enfants
> - Peut bloquer des configurations légitimes
> - Complique le dépannage
> 
> **Recommandation :** Réservé aux exigences de sécurité ou conformité critiques.

#### Activer Enforced

**Via GPMC :**

```
1. Ouvrir GPMC
2. Naviguer vers l'OU où la GPO est liée
3. Clic droit sur le lien GPO → "Enforced"
4. La coche apparaît et l'icône devient un cadenas
```

**Via PowerShell :**

```powershell
# Activer Enforced sur un lien existant
Set-GPLink -Name "Politique Sécurité Critique" `
           -Target "DC=entreprise,DC=local" `
           -Enforced Yes

# Créer un lien avec Enforced directement
New-GPLink -Name "Antivirus Obligatoire" `
           -Target "OU=Workstations,DC=entreprise,DC=local" `
           -Enforced Yes

# Désactiver Enforced
Set-GPLink -Name "Politique Sécurité Critique" `
           -Target "DC=entreprise,DC=local" `
           -Enforced No

# Vérifier le statut Enforced
Get-GPInheritance -Target "OU=IT,DC=entreprise,DC=local" |
    Select-Object -ExpandProperty GpoLinks |
    Where-Object {$_.Enforced -eq $true}
```

#### Comportement avec Enforced

**Sans Enforced :**

```
Domaine
├── GPO-Domaine: USB = Blocked
│
└── OU=IT
    └── GPO-IT: USB = Allowed

Résultat IT: USB = Allowed (GPO-IT gagne, plus proche)
```

**Avec Enforced :**

```
Domaine
├── GPO-Domaine: USB = Blocked [ENFORCED] 🔒
│
└── OU=IT
    └── GPO-IT: USB = Allowed

Résultat IT: USB = Blocked (GPO-Domaine forcée malgré GPO-IT)
```

> [!tip] Ordre de Priorité avec Enforced
> **Règle absolue :**
> 
> 1. GPO Enforced (de la plus proche à la plus éloignée)
> 2. GPO normales (de la plus proche à la plus éloignée)
> 
> **Plusieurs GPO Enforced :**
> - GPO Enforced d'OU enfant > GPO Enforced d'OU parent
> - Mais toutes les Enforced > toutes les non-Enforced

### Block Inheritance (Bloquer l'Héritage)

> [!info] Block Inheritance
> **Objectif :** Empêcher les GPO des niveaux supérieurs (domaine, OU parent) de s'appliquer
> 
> **Utilisation :** Créer une "zone autonome" avec ses propres paramètres
> 
> **Effet :** Les GPO parentes ne sont PAS héritées (sauf si Enforced)
> 
> **Symbole GPMC :** Icône de bouclier bleu avec exclamation 🛡️

#### Quand Utiliser Block Inheritance ?

**Scénarios typiques :**

| Scénario | Raison |
|----------|--------|
| OU=Serveurs-DMZ | Configuration sécurité très différente du reste |
| OU=Kiosques-Public | Environnement verrouillé unique |
| OU=Laboratoire-Test | Besoin de liberté pour tests sans GPO entreprise |
| OU=Direction | Paramètres spéciaux sans restrictions standard |

> [!warning] Attention aux Implications
> **Block Inheritance peut causer :**
> - Perte de paramètres de sécurité importants
> - Non-application de politiques entreprise obligatoires
> - Difficultés de support (configuration non standard)
> 
> **⚠️ Les GPO "Enforced" ne sont PAS bloquées !**

#### Activer Block Inheritance

**Via GPMC :**

```
1. Ouvrir GPMC
2. Clic droit sur l'OU → "Block Inheritance"
3. Une coche apparaît et l'icône bouclier s'affiche
```

**Via PowerShell :**

```powershell
# Activer Block Inheritance sur une OU
Set-GPInheritance -Target "OU=DMZ,OU=Servers,DC=entreprise,DC=local" `
                  -IsBlocked Yes

# Désactiver Block Inheritance
Set-GPInheritance -Target "OU=DMZ,OU=Servers,DC=entreprise,DC=local" `
                  -IsBlocked No

# Vérifier le statut
Get-GPInheritance -Target "OU=DMZ,OU=Servers,DC=entreprise,DC=local" |
    Select-Object Path, GpoInheritanceBlocked
```

#### Comportement avec Block Inheritance

**Sans Block Inheritance :**

```
Domaine
├── GPO-Domaine: Fond d'écran = logo-entreprise.jpg
│
└── OU=IT
    ├── GPO-IT-Services: Lecteur Y = \\serveur\it
    │
    └── OU=IT-Admins
        └── GPO-IT-Admins: AccèsPanneauConfig = Enabled

Résultat IT-Admins:
✅ Fond d'écran = logo-entreprise.jpg (hérité domaine)
✅ Lecteur Y (hérité IT)
✅ Accès panneau config (GPO locale)
```

**Avec Block Inheritance sur OU=IT-Admins :**

```
Domaine
├── GPO-Domaine: Fond d'écran = logo-entreprise.jpg
│
└── OU=IT
    ├── GPO-IT-Services: Lecteur Y = \\serveur\it
    │
    └── OU=IT-Admins [BLOCK INHERITANCE] 🛡️
        └── GPO-IT-Admins: AccèsPanneauConfig = Enabled

Résultat IT-Admins:
❌ Fond d'écran = Par défaut (GPO-Domaine bloquée)
❌ Lecteur Y = Non mappé (GPO-IT bloquée)
✅ Accès panneau config (GPO locale)
```

### Interaction Enforced vs Block Inheritance

> [!important] Règle d'Or
> **Enforced bat toujours Block Inheritance**
> 
> Une GPO marquée "Enforced" passe à travers "Block Inheritance".

**Exemple complet :**

```
Domaine
├── GPO-Sécurité: USB = Blocked [ENFORCED] 🔒
├── GPO-Domaine: Fond d'écran = logo.jpg
│
└── OU=IT
    ├── GPO-IT: Lecteur Y = \\serveur\it
    │
    └── OU=IT-Tests [BLOCK INHERITANCE] 🛡️
        └── GPO-IT-Tests: Logiciels = Tests autorisés

Résultat IT-Tests:
✅ USB = Blocked (GPO-Sécurité ENFORCED passe le blocage)
❌ Fond d'écran = Par défaut (GPO-Domaine bloquée)
❌ Lecteur Y = Non mappé (GPO-IT bloquée)
✅ Logiciels tests (GPO locale)
```

**Matrice de décision :**

| GPO Parent | OU Enfant a Block Inheritance | Résultat |
|------------|-------------------------------|----------|
| GPO normale | Non | ✅ GPO héritée |
| GPO normale | Oui | ❌ GPO bloquée |
| GPO Enforced | Non | ✅ GPO héritée |
| GPO Enforced | Oui | ✅ GPO héritée quand même ! |

### Dépannage Enforced/Block

```powershell
# Script de diagnostic héritage
function Get-GPOInheritanceReport {
    param([string]$TargetOU)
    
    $inheritance = Get-GPInheritance -Target $TargetOU
    
    Write-Host "`n=== GPO Inheritance Report ===" -ForegroundColor Cyan
    Write-Host "OU: $TargetOU" -ForegroundColor Yellow
    Write-Host "Block Inheritance: $($inheritance.GpoInheritanceBlocked)" -ForegroundColor $(if($inheritance.GpoInheritanceBlocked){"Red"}else{"Green"})
    
    Write-Host "`nGPO Links (ordre d'application):" -ForegroundColor Cyan
    $inheritance.GpoLinks | ForEach-Object {
        $color = "White"
        $status = ""
        
        if ($_.Enforced) {
            $color = "Magenta"
            $status = "[ENFORCED]"
        }
        if (-not $_.Enabled) {
            $color = "Gray"
            $status += "[DISABLED]"
        }
        
        Write-Host "  $($_.Order). $($_.DisplayName) $status" -ForegroundColor $color
    }
}

# Utilisation
Get-GPOInheritanceReport -TargetOU "OU=IT-Admins,OU=IT,DC=entreprise,DC=local"
```

---

## 🔒 Filtrage de Sécurité

Le **filtrage de sécurité** permet de contrôler précisément quels utilisateurs, groupes ou ordinateurs reçoivent une GPO, même s'ils sont dans une OU où la GPO est liée.

### Concept du Filtrage de Sécurité

> [!info] Filtrage de Sécurité
> **Principe :** Une GPO s'applique uniquement si l'objet (utilisateur/ordinateur) dispose des permissions "Read" ET "Apply Group Policy" sur la GPO.
> 
> **Par défaut :** Toute GPO donne ces permissions au groupe "Authenticated Users"
> 
> **Filtrage :** Modifier ces permissions pour restreindre l'application

```
OU=IT (100 utilisateurs)
└── GPO-Logiciels-Admins (liée à OU=IT)
    
Sans filtrage: 
✅ Les 100 utilisateurs reçoivent la GPO

Avec filtrage sur groupe "IT-Admins" (10 utilisateurs):
✅ 10 utilisateurs IT-Admins reçoivent la GPO
❌ 90 autres utilisateurs ne la reçoivent pas
```

### Permissions Requises pour Application GPO

> [!info] Permissions Nécessaires
> Pour qu'une GPO s'applique, l'objet doit avoir :
> 
> 1. **Read** : Lire le contenu de la GPO
> 2. **Apply Group Policy** : Permission spéciale d'application
> 
> **Par défaut :**
> - "Authenticated Users" : Read + Apply Group Policy
> - "SYSTEM" : Full Control
> - "Domain Admins" : Full Control
> - "Enterprise Admins" : Full Control

```powershell
# Voir les permissions d'une GPO
Get-GPPermission -Name "Logiciels Standard" -All

# Résultat type:
# Trustee: Authenticated Users
# Permission: GpoRead, GpoApply
```

### Configurer le Filtrage via GPMC

**Dans la console GPMC :**

```
1. Ouvrir GPMC
2. Sélectionner la GPO
3. Onglet "Scope" → Section "Security Filtering"
4. Par défaut: "Authenticated Users"

Pour filtrer:
5. Remove "Authenticated Users"
6. Add → Sélectionner groupe ou utilisateur spécifique
7. OK
```

> [!warning] Ne Jamais Retirer les Permissions Système
> **Toujours laisser :**
> - SYSTEM
> - Domain Admins
> - Enterprise Admins
> 
> Ces comptes ont besoin de "Read" pour gérer les GPO.
> Retirer = corruption de GPO ou impossibilité de gestion.

### Configurer le Filtrage via PowerShell

```powershell
# Retirer Authenticated Users
Set-GPPermission -Name "GPO-Logiciels-Admins" `
                 -TargetName "Authenticated Users" `
                 -TargetType Group `
                 -PermissionLevel None

# Ajouter un groupe spécifique avec droits Apply
Set-GPPermission -Name "GPO-Logiciels-Admins" `
                 -TargetName "GRP-IT-Admins" `
                 -TargetType Group `
                 -PermissionLevel GpoApply

# Ajouter plusieurs groupes
$groupes = @("GRP-IT-Admins", "GRP-IT-Managers", "GRP-DevOps")
foreach ($groupe in $groupes) {
    Set-GPPermission -Name "GPO-Logiciels-Admins" `
                     -TargetName $groupe `
                     -TargetType Group `
                     -PermissionLevel GpoApply
}

# Ajouter un ordinateur spécifique
Set-GPPermission -Name "GPO-Config-Serveur-Test" `
                 -TargetName "SRV-TEST-01$" `
                 -TargetType Computer `
                 -PermissionLevel GpoApply
```

### Niveaux de Permission

| Niveau | Read | Apply | Edit | Utilisation |
|--------|------|-------|------|-------------|
| **None** | ❌ | ❌ | ❌ | Aucun accès |
| **GpoRead** | ✅ | ❌ | ❌ | Voir mais pas appliquer |
| **GpoApply** | ✅ | ✅ | ❌ | Application normale |
| **GpoEdit** | ✅ | ✅ | ✅ | Modification possible |
| **GpoEditDeleteModifySecurity** | ✅ | ✅ | ✅ | Contrôle total |

```powershell
# Donner uniquement Read (pour audit sans application)
Set-GPPermission -Name "GPO-Audit" `
                 -TargetName "GRP-Auditeurs" `
                 -TargetType Group `
                 -PermissionLevel GpoRead

# Donner droits d'édition (délégation)
Set-GPPermission -Name "GPO-IT-Config" `
                 -TargetName "GRP-IT-Admins" `
                 -TargetType Group `
                 -PermissionLevel GpoEdit
```

### Filtrage Négatif (Deny)

> [!warning] Deny - Utiliser avec Extrême Prudence
> **Deny** a la priorité absolue sur toutes les permissions "Allow".
> 
> **Problème :** Si un utilisateur est dans un groupe avec Deny, il ne recevra pas la GPO même s'il est dans d'autres groupes avec Allow.

```powershell
# Ajouter une permission Deny (rarement recommandé)
Set-GPPermission -Name "GPO-Logiciels" `
                 -TargetName "GRP-Stagiaires" `
                 -TargetType Group `
                 -PermissionLevel None `
                 -Replace

# Attention: Deny explicite
# Méthode non recommandée, préférer le filtrage positif
```

> [!tip] Préférer le Filtrage Positif
> **Recommandation :**
> - ✅ Spécifier QUI reçoit la GPO (groupes avec GpoApply)
> - ❌ Éviter de spécifier qui NE reçoit PAS (Deny)
> 
> **Raison :** Plus simple à gérer et à auditer.

### Cas d'Usage du Filtrage

#### Scénario 1 : GPO pour Groupe Spécifique

**Besoin :** Installer des outils d'administration uniquement pour les IT Admins

```powershell
# Créer la GPO
New-GPO -Name "GPO-Outils-IT-Admins"

# Configurer le contenu de la GPO (via GPMC ou scripts)
# ...

# Lier à l'OU IT
New-GPLink -Name "GPO-Outils-IT-Admins" `
           -Target "OU=IT,DC=entreprise,DC=local"

# Configurer le filtrage de sécurité
# Retirer Authenticated Users
Set-GPPermission -Name "GPO-Outils-IT-Admins" `
                 -TargetName "Authenticated Users" `
                 -TargetType Group `
                 -PermissionLevel None

# Ajouter le groupe IT Admins
Set-GPPermission -Name "GPO-Outils-IT-Admins" `
                 -TargetName "GRP-IT-Admins" `
                 -TargetType Group `
                 -PermissionLevel GpoApply
```

**Résultat :**
- GPO liée à OU=IT (tous les utilisateurs IT)
- Mais seuls les membres de GRP-IT-Admins la reçoivent

#### Scénario 2 : GPO pour Ordinateurs Spécifiques

**Besoin :** Configuration spéciale pour serveurs de production uniquement

```powershell
New-GPO -Name "GPO-Serveurs-Production"

New-GPLink -Name "GPO-Serveurs-Production" `
           -Target "OU=Servers,DC=entreprise,DC=local"

# Filtrer sur groupe contenant les serveurs de prod
Set-GPPermission -Name "GPO-Serveurs-Production" `
                 -TargetName "Authenticated Users" `
                 -TargetType Group `
                 -PermissionLevel None

Set-GPPermission -Name "GPO-Serveurs-Production" `
                 -TargetName "GRP-Serveurs-Production" `
                 -TargetType Group `
                 -PermissionLevel GpoApply
```

> [!tip] Groupes de Sécurité pour Filtrage
> **Bonne pratique :** Créer des groupes de sécurité spécifiquement pour le filtrage GPO.
> 
> **Convention de nommage :**
> - `GPO-[NomGPO]-Apply`
> - Exemple: `GPO-Logiciels-IT-Apply`
> 
> **Avantages :**
> - Clair et documenté
> - Facilite la gestion
> - Audit simplifié

### Vérifier le Filtrage

```powershell
# Voir toutes les permissions d'une GPO
Get-GPPermission -Name "GPO-Logiciels-Admins" -All |
    Select-Object Trustee, Permission

# Voir uniquement ceux qui peuvent l'appliquer
Get-GPPermission -Name "GPO-Logiciels-Admins" -All |
    Where-Object {$_.Permission -like "*GpoApply*"} |
    Select-Object Trustee

# Vérifier si un utilisateur/groupe spécifique peut l'appliquer
Get-GPPermission -Name "GPO-Logiciels-Admins" `
                 -TargetName "GRP-IT-Admins" `
                 -TargetType Group
```

### Combinaison Filtrage + Héritage

> [!info] Filtrage ET Héritage Ensemble
> Le filtrage de sécurité et l'héritage fonctionnent ensemble :
> 
> 1. **Héritage détermine quelles GPO atteignent un niveau**
> 2. **Filtrage détermine qui les reçoit à ce niveau**

**Exemple combiné :**

```
Domaine
├── GPO-Domaine: Antivirus (Authenticated Users) ← Tous
│
└── OU=Services
    ├── GPO-Services: Lecteurs (Authenticated Users) ← Tous dans Services
    │
    └── OU=IT
        ├── GPO-IT: Logiciels (GRP-IT seulement) ← Filtré
        │
        └── OU=IT-Admins [BLOCK INHERITANCE]
            └── GPO-Admins: Outils (GRP-IT-Admins seulement) ← Filtré

Utilisateur: Jean (membre GRP-IT-Admins) dans OU=IT-Admins

Héritage normal:
✅ GPO-Domaine (hérité domaine)
✅ GPO-Services (hérité Services)
✅ GPO-IT (hérité IT)
✅ GPO-Admins (lié direct)

Avec Block Inheritance sur IT-Admins:
❌ GPO-Domaine (bloqué)
❌ GPO-Services (bloqué)
❌ GPO-IT (bloqué)
✅ GPO-Admins (lié direct)

Avec Filtrage:
Jean est dans GRP-IT-Admins → ✅ Reçoit GPO-Admins
```

---

## 🛠️ Outils de Gestion (GPMC)

La **Group Policy Management Console (GPMC)** est l'outil central de gestion des stratégies de groupe dans Active Directory.

### Installation de GPMC

> [!info] Disponibilité GPMC
> **Pré-installé sur :**
> - Windows Server (toutes versions depuis 2008 R2)
> - Contrôleurs de domaine
> 
> **À installer sur :**
> - Windows 10/11 Pro/Enterprise
> - Via RSAT (Remote Server Administration Tools)

#### Installation sur Windows 10/11

**Méthode 1 : Via Paramètres (Windows 10 1809+)**

```powershell
# Installer RSAT: Group Policy Management Tools
Add-WindowsCapability -Online -Name "Rsat.GroupPolicy.Management.Tools~~~~0.0.1.0"

# Vérifier l'installation
Get-WindowsCapability -Online -Name "Rsat.GroupPolicy.Management.Tools*"
```

**Méthode 2 : Via Interface Graphique**

```
1. Paramètres → Applications → Fonctionnalités facultatives
2. Ajouter une fonctionnalité
3. Rechercher "RSAT: Group Policy Management Tools"
4. Installer
```

**Méthode 3 : Téléchargement RSAT (anciennes versions Windows)**

```
1. Télécharger RSAT depuis Microsoft Download Center
2. Installer le package
3. Activer "Group Policy Management" dans:
   Panneau de configuration → Programmes → Activer/Désactiver fonctionnalités Windows
```

### Lancement de GPMC

```powershell
# Lancer GPMC
gpmc.msc

# Ou depuis le menu Démarrer
# Outils d'administration → Gestion des stratégies de groupe
```

### Structure de GPMC

```
Group Policy Management
│
├── Forest: entreprise.local
│   │
│   ├── Domains
│   │   └── entreprise.local
│   │       ├── Default Domain Policy
│   │       ├── Default Domain Controllers Policy
│   │       │
│   │       ├── OU=Paris
│   │       │   ├── GPO liées affichées ici
│   │       │   └── OU enfants...
│   │       │
│   │       └── Group Policy Objects
│   │           ├── [Toutes les GPO du domaine]
│   │           ├── GPO-Sécurité
│   │           ├── GPO-Logiciels
│   │           └── ...
│   │
│   ├── Sites
│   │   └── [Sites AD si GPO liées]
│   │
│   ├── Group Policy Modeling (simulation)
│   └── Group Policy Results (résultats réels)
│
└── WMI Filters
```

### Fonctionnalités Principales GPMC

#### 1. Créer une GPO

**Via interface :**

```
1. Développer Domains → votre domaine → Group Policy Objects
2. Clic droit sur "Group Policy Objects" → New
3. Entrer le nom: "GPO-Configuration-IT"
4. OK
```

**Via PowerShell :**

```powershell
# Créer une GPO
New-GPO -Name "GPO-Configuration-IT" -Comment "Configuration pour le département IT"

# Créer et lier en une opération
New-GPO -Name "GPO-Securite-Workstations" | 
    New-GPLink -Target "OU=Workstations,DC=entreprise,DC=local"
```

#### 2. Éditer une GPO

**Via interface :**

```
1. Développer Group Policy Objects
2. Clic droit sur la GPO → Edit
3. Group Policy Management Editor s'ouvre
4. Naviguer dans Computer/User Configuration
5. Modifier les paramètres souhaités
```

**Structure de l'éditeur :**

```
GPO Editor
│
├── Computer Configuration
│   ├── Policies
│   │   ├── Software Settings
│   │   ├── Windows Settings
│   │   │   ├── Security Settings ← Politiques sécurité
│   │   │   │   ├── Account Policies
│   │   │   │   │   ├── Password Policy
│   │   │   │   │   └── Account Lockout Policy
│   │   │   │   ├── Local Policies
│   │   │   │   ├── Windows Firewall
│   │   │   │   └── ...
│   │   │   └── Scripts
│   │   └── Administrative Templates ← Modèles admin
│   │       ├── Control Panel
│   │       ├── Network
│   │       ├── System
│   │       └── Windows Components
│   │
│   └── Preferences ← Préférences (configuration item)
│       ├── Windows Settings
│       ├── Control Panel Settings
│       └── ...
│
└── User Configuration
    └── [Structure similaire]
```

> [!tip] Policies vs Preferences
> **Policies (Stratégies) :**
> - Obligatoires et verrouillées
> - Utilisateur ne peut pas modifier
> - Réappliquées régulièrement
> 
> **Preferences (Préférences) :**
> - Paramètres "souples"
> - Utilisateur peut modifier après application
> - Appliquées une fois par défaut

#### 3. Sauvegarder une GPO

```powershell
# Créer dossier de sauvegarde
New-Item -Path "C:\GPO-Backups" -ItemType Directory -Force

# Sauvegarder une GPO spécifique
Backup-GPO -Name "GPO-Sécurité-Critique" -Path "C:\GPO-Backups"

# Sauvegarder toutes les GPO
Backup-GPO -All -Path "C:\GPO-Backups"

# Sauvegarder avec commentaire
Backup-GPO -Name "Default Domain Policy" `
           -Path "C:\GPO-Backups" `
           -Comment "Sauvegarde avant modification politique mots de passe"

# Sauvegarder par GUID
Backup-GPO -Guid "12345678-1234-1234-1234-123456789ABC" -Path "C:\GPO-Backups"
```

**Via GPMC interface :**

```
1. Clic droit sur GPO → Back Up...
2. Choisir emplacement
3. Ajouter description (optionnel)
4. Back Up
```

> [!tip] Fréquence de Sauvegarde
> **Recommandations :**
> - Avant toute modification majeure d'une GPO
> - Sauvegarde planifiée hebdomadaire/mensuelle de toutes les GPO
> - Avant mise à jour de serveur ou changement infrastructure
> - Conserver plusieurs versions historiques

#### 4. Restaurer une GPO

```powershell
# Lister les sauvegardes disponibles
Get-GPOBackup -Path "C:\GPO-Backups"

# Restaurer la dernière sauvegarde d'une GPO
Restore-GPO -Name "GPO-Sécurité-Critique" -Path "C:\GPO-Backups"

# Restaurer une sauvegarde spécifique par ID
Restore-GPO -BackupId "12345678-ABCD-1234-5678-123456789ABC" -Path "C:\GPO-Backups"

# Restaurer toutes les GPO
Get-GPOBackup -Path "C:\GPO-Backups" | ForEach-Object {
    Restore-GPO -BackupId $_.Id -Path "C:\GPO-Backups"
}
```

#### 5. Copier/Importer une GPO

```powershell
# Copier une GPO (crée une nouvelle GPO)
Copy-GPO -SourceName "GPO-Template-Workstation" `
         -TargetName "GPO-Workstation-Finance"

# Importer paramètres depuis sauvegarde vers GPO existante
Import-GPO -BackupId "12345678-ABCD-1234-5678-123456789ABC" `
           -Path "C:\GPO-Backups" `
           -TargetName "GPO-Nouvelle" `
           -CreateIfNeeded

# Importer vers nouvelle GPO
$newGPO = New-GPO -Name "GPO-Importée"
Import-GPO -BackupGpoName "GPO-Source" `
           -Path "C:\GPO-Backups" `
           -TargetName "GPO-Importée"
```

> [!info] Copy vs Import
> **Copy-GPO :**
> - Copie une GPO existante vers nouvelle GPO
> - Dans le même domaine
> - Conserve tous les paramètres
> 
> **Import-GPO :**
> - Importe depuis une sauvegarde
> - Peut être cross-domaine/forêt
> - Nécessite migration tables si changement domaine

#### 6. Générer des Rapports

```powershell
# Rapport HTML d'une GPO
Get-GPOReport -Name "GPO-Sécurité" `
              -ReportType Html `
              -Path "C:\Reports\GPO-Sécurité.html"

# Rapport XML (pour traitement automatisé)
Get-GPOReport -Name "GPO-Sécurité" `
              -ReportType Xml `
              -Path "C:\Reports\GPO-Sécurité.xml"

# Rapport de toutes les GPO
Get-GPO -All | ForEach-Object {
    $reportPath = "C:\Reports\$($_.DisplayName).html"
    Get-GPOReport -Guid $_.Id -ReportType Html -Path $reportPath
}

# Rapport résumé de toutes les GPO (CSV)
Get-GPO -All | 
    Select-Object DisplayName, GpoStatus, CreationTime, ModificationTime, @{
        Name="LinksCount"
        Expression={(Get-GPOReport -Guid $_.Id -ReportType Xml | Select-Xml -XPath "//LinksTo").Count}
    } |
    Export-Csv -Path "C:\Reports\GPO-Summary.csv" -NoTypeInformation
```

**Via GPMC :**

```
1. Clic droit sur GPO → Settings
2. Rapport HTML s'affiche dans le panneau
3. Ou: Save Report → Choisir HTML ou XML
```

#### 7. Rechercher des Paramètres

**Via GPMC :**

```
1. Clic droit sur domaine ou "Group Policy Objects"
2. Search...
3. Entrer critères:
   - Nom de GPO
   - Paramètre spécifique
   - GUID
4. Search
```

```powershell
# Rechercher GPO contenant un terme
Get-GPO -All | Where-Object {$_.DisplayName -like "*Sécurité*"}

# Rechercher GPO modifiées récemment (7 derniers jours)
$date = (Get-Date).AddDays(-7)
Get-GPO -All | Where-Object {$_.ModificationTime -gt $date} |
    Select-Object DisplayName, ModificationTime

# Rechercher paramètre spécifique dans toutes les GPO (complexe)
# Nécessite analyse des XML
Get-GPO -All | ForEach-Object {
    $report = Get-GPOReport -Guid $_.Id -ReportType Xml
    if ($report -match "DisableUSBStorage") {
        [PSCustomObject]@{
            GPOName = $_.DisplayName
            Found = $true
        }
    }
}
```

### Group Policy Modeling (Simulation)

> [!info] Group Policy Modeling
> **Objectif :** Simuler l'application de GPO **sans vraiment les appliquer**
> 
> **Utilité :**
> - Tester avant déploiement
> - Comprendre quelles GPO s'appliqueraient
> - Planifier des changements d'OU
> - Formation et démonstration

```
Dans GPMC:
1. Clic droit sur "Group Policy Modeling"
2. Group Policy Modeling Wizard
3. Sélectionner:
   - OU utilisateur
   - OU ordinateur
   - Groupes d'appartenance (optionnel)
   - Site (optionnel)
   - Filtres WMI (optionnel)
4. Finish
5. Résultat montre GPO qui s'appliqueraient
```

**Via PowerShell :**

```powershell
# Impossible nativement via cmdlets PowerShell
# Nécessite GPMC COM objects ou interface graphique
```

### Group Policy Results (Résultats Réels)

> [!info] Group Policy Results
> **Objectif :** Voir les GPO **réellement appliquées** à un utilisateur/ordinateur
> 
> **Différence avec Modeling :**
> - Results = Ce qui EST appliqué (réel)
> - Modeling = Ce qui SERAIT appliqué (simulation)

```
Dans GPMC:
1. Clic droit sur "Group Policy Results"
2. Group Policy Results Wizard
3. Sélectionner ordinateur cible
4. Sélectionner utilisateur sur cet ordinateur
5. Finish
6. Résultat montre GPO appliquées avec détails
```

**Via gpresult (en ligne de commande sur le poste) :**

```powershell
# Résultats en mode texte
gpresult /r

# Par portée (ordinateur seulement)
gpresult /r /scope:computer

# Par portée (utilisateur seulement)
gpresult /r /scope:user

# Rapport HTML complet
gpresult /h C:\Temp\GPResult.html

# Rapport pour utilisateur/ordinateur distant
gpresult /s NomOrdinateur /u Domaine\Utilisateur /p MotDePasse /h C:\Temp\Remote-GPResult.html

# Avec verbose (tous les détails)
gpresult /v

# Super verbose (maximum d'infos)
gpresult /z
```

**Interpréter gpresult :**

```
COMPUTER SETTINGS
-----------------
Last time Group Policy was applied: 09/02/2025 à 10:30:15
Group Policy was applied from: dc01.entreprise.local
Group Policy slow link threshold: 500 kbps
Domain Name: ENTREPRISE
Domain Type: Windows 2008 or later

Applied Group Policy Objects
-----------------------------
Default Domain Policy               ← GPO appliquée
GPO-Sécurité-Workstations          ← GPO appliquée
GPO-Config-IT                       ← GPO appliquée

The following GPOs were not applied because they were filtered out
-------------------------------------------------------------------
GPO-Logiciels-Finance               ← Filtrée (pas de permission)
    Filtering: Denied (Security)
```

### Délégation de Gestion GPO

```powershell
# Déléguer la création de GPO
Set-GPPermission -Name "GPO-IT-Config" `
                 -TargetName "GRP-IT-Managers" `
                 -TargetType Group `
                 -PermissionLevel GpoEdit

# Déléguer la liaison de GPO sur une OU
# Via GPMC: Clic droit OU → Delegate Control → Group Policy Objects

# Donner droits de création de GPO dans le domaine
# Via GPMC: Clic droit "Group Policy Objects" → Delegation
```

### Maintenance et Optimisation

```powershell
# Supprimer les GPO non liées (orphelines)
Get-GPO -All | Where-Object {
    $gpo = $_
    $links = ($gpo | Get-GPOReport -ReportType Xml | Select-Xml -XPath "//LinksTo").Count
    $links -eq 0
} | ForEach-Object {
    Write-Host "GPO non liée: $($_.DisplayName)" -ForegroundColor Yellow
    # Décommenter pour supprimer réellement
    # Remove-GPO -Guid $_.Id -Confirm:$false
}

# Identifier GPO avec configuration vide
Get-GPO -All | Where-Object {
    $_.User.Enabled -eq $false -and $_.Computer.Enabled -eq $false
} | Select-Object DisplayName, CreationTime

# Audit des GPO désactivées
Get-GPO -All | Where-Object {$_.GpoStatus -eq "AllSettingsDisabled"} |
    Select-Object DisplayName, GpoStatus, ModificationTime
```

### Scripts Utiles GPMC

```powershell
# Rapport complet de santé GPO
function Get-GPOHealthReport {
    $gpos = Get-GPO -All
    $report = @()
    
    foreach ($gpo in $gpos) {
        $xml = Get-GPOReport -Guid $gpo.Id -ReportType Xml
        $links = (Select-Xml -Xml ([xml]$xml) -XPath "//LinksTo").Count
        
        $report += [PSCustomObject]@{
            Name = $gpo.DisplayName
            Status = $gpo.GpoStatus
            Created = $gpo.CreationTime
            Modified = $gpo.ModificationTime
            UserEnabled = $gpo.User.Enabled
            ComputerEnabled = $gpo.Computer.Enabled
            LinksCount = $links
            Owner = $gpo.Owner
        }
    }
    
    $report | Export-Csv -Path "C:\Reports\GPO-Health-$(Get-Date -Format 'yyyyMMdd').csv" -NoTypeInformation
    $report | Format-Table -AutoSize
}

# Exécution
Get-GPOHealthReport
```

---

## 🎯 Récapitulatif

> [!success] Points Clés à Retenir
> 
> **Concept GPO :**
> - Collections de paramètres appliqués automatiquement
> - Computer Configuration (au démarrage) vs User Configuration (à la connexion)
> - Centralisation, cohérence, automatisation
> 
> **Structure :**
> - GPC dans Active Directory (métadonnées)
> - GPT dans SYSVOL (fichiers de config)
> - Liés par GUID unique
> - Synchronisation GPC/GPT critique
> 
> **Liaison :**
> - Une GPO peut être liée à domaine, site, OU
> - Une GPO peut avoir multiples liens
> - Ordre de liaison détermine priorité locale
> - Lien désactivé ≠ GPO supprimée
> 
> **Héritage :**
> - LSDOU : Local → Site → Domain → OU
> - Héritage du parent vers l'enfant
> - Dernière GPO appliquée gagne (sauf Enforced)
> - Computer Config et User Config traités séparément
> 
> **Enforced et Block Inheritance :**
> - Enforced : GPO prioritaire absolue, passe tout
> - Block Inheritance : Bloque GPO parentes (sauf Enforced)
> - Enforced bat Block Inheritance toujours
> 
> **Filtrage de Sécurité :**
> - Contrôle précis : qui reçoit la GPO
> - Basé sur permissions Read + Apply Group Policy
> - Retirer Authenticated Users, ajouter groupes spécifiques
> - Filtrage positif préféré à Deny
> 
> **GPMC :**
> - Outil central de gestion GPO
> - Création, édition, liaison, sauvegarde, restauration
> - Modeling (simulation) vs Results (réel)
> - gpresult pour diagnostic sur poste

---

> [!quote] Bonne Pratique Finale
> **"La maîtrise des GPO repose sur la compréhension de l'héritage, du filtrage et de la délégation. Documentez votre architecture GPO, testez avec Modeling avant de déployer, et sauvegardez systématiquement avant toute modification. Les GPO sont puissantes mais exigent rigueur et méthode."**
