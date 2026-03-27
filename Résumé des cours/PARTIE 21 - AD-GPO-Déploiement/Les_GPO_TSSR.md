# Les GPO (Group Policy Objects)

## Document de révision TSSR - Titre RNCP

---

**Formation** : Technicien Supérieur Systèmes et Réseaux (TSSR)

**Sujet** : Les stratégies de groupe (GPO)

**Date** : Janvier 2026

**Type** : Synthèse de cours complète

---

## 📋 Sommaire

1. [[#Introduction|Introduction]]
2. [[#Définition|Définition]]
   - [[#Qu'est-ce qu'une GPO ?|Qu'est-ce qu'une GPO ?]]
   - [[#Objectifs des GPO|Objectifs des GPO]]
   - [[#OS pris en compte|OS pris en compte]]
   - [[#Constitution d'une GPO|Constitution d'une GPO]]
   - [[#États des GPO|États des GPO]]
3. [[#Règles de priorité|Règles de priorité]]
   - [[#Stratégies locales|Stratégies locales]]
   - [[#Stratégies locales vs GPO|Stratégies locales vs GPO]]
   - [[#Priorité LSDOU|Priorité LSDOU]]
   - [[#Héritage des GPO|Héritage des GPO]]
   - [[#Fonctionnement sur une OU|Fonctionnement sur une OU]]
   - [[#Filtrage de sécurité|Filtrage de sécurité]]
4. [[#Bonnes pratiques|Bonnes pratiques]]
5. [[#Points clés à retenir|Points clés à retenir]]
6. [[#Glossaire technique|Glossaire technique]]
7. [[#Liens et ressources|Liens et ressources]]

---

## Introduction

> [!abstract] Vue d'ensemble
> Les **GPO** (Group Policy Objects) sont l'un des outils les plus puissants d'Active Directory pour gérer de manière centralisée la configuration et la sécurité d'un parc informatique Windows.

> [!question] Que peut-on gérer avec les GPO ?
> Les GPO permettent de contrôler pratiquement tous les aspects de la configuration des ordinateurs et des utilisateurs dans un environnement Active Directory :
> - Configuration de la sécurité
> - Gestion de l'interface utilisateur
> - Déploiement de logiciels
> - Scripts d'ouverture/fermeture de session
> - Redirection de dossiers
> - Paramètres réseau
> - Et bien plus encore...

---

## Définition

---

## Qu'est-ce qu'une GPO ?

> [!quote] Définition
> Les **objets de stratégies de groupes**, ou **GPO** (Group Policy Object), sont des **collections virtuelles de politiques de sécurité** qui permettent de gérer avec une méthode centralisée un parc informatique.

### Caractéristiques d'une GPO

> [!info] Propriétés fondamentales
> Une GPO :
> - Possède un **nom unique** (identifié par un GUID)
> - Est un **objet Active Directory**
> - Contient des **paramètres de configuration**
> - S'applique aux **utilisateurs et/ou ordinateurs**
> - Est **répliquée** entre les contrôleurs de domaine

### Capacités de gestion

> [!important] Domaines de gestion des GPO
> Les GPO permettent de gérer :
> 
> **1. Gestion des ordinateurs et des utilisateurs** :
> - Configuration du système d'exploitation
> - Paramètres d'environnement
> - Préférences utilisateur
> 
> **2. Politiques de sécurité** :
> - Restrictions d'utilisation
> - Contrôle d'accès
> - Politiques de mots de passe
> - Verrouillage de compte
> 
> **3. Gestion de l'interface graphique** :
> - Personnalisation du bureau
> - Masquage d'éléments du menu Démarrer
> - Restrictions d'accès aux paramètres
> 
> **4. Déploiement** :
> - Installation de logiciels
> - Déploiement de scripts
> - Configuration de services Windows
> 
> **5. Scripts de connexion** :
> - Scripts d'ouverture de session (logon)
> - Scripts de fermeture de session (logoff)
> - Scripts de démarrage/arrêt ordinateur
> 
> **6. Redirection de dossiers** :
> - Documents, Bureau, Images, etc.
> - Vers des emplacements réseau centralisés

---

## Objectifs des GPO

> [!success] Méthode de gestion centralisée
> Les GPO sont une **méthode de gestion de configuration de parc informatique** qui permet de :
> - **Définir une configuration cible** de sécurité
> - **Standardiser les installations** sur l'ensemble du parc
> - **Appliquer des politiques** de manière cohérente
> - **Automatiser** la configuration des postes et serveurs
> - **Réduire les interventions manuelles** des administrateurs

---

## OS pris en compte

> [!info] Compatibilité des GPO
> Les GPO sont **fonctionnelles sur les ordinateurs ayant un OS Microsoft** (client ou serveur) :
> - Windows Client : Windows 7, 8, 10, 11
> - Windows Server : 2008, 2012, 2016, 2019, 2022

### GPO et Linux

> [!warning] Support limité pour Linux
> Il existe des **implémentations très partielles** de clients GPO pour les environnements Linux, mais :
> 
> **Limitations** :
> - La **plupart des modèles de GPO** proposés dans la console de Gestion des Stratégies de Groupe ne seront **pas pris en compte** par les clients GPO Linux
> - Si l'on affecte une GPO à un client qui ne peut pas l'interpréter, la GPO sera alors **ignorée**
> 
> **Solutions alternatives pour Linux** :
> - Configuration manuelle
> - Outils de gestion de configuration (Ansible, Puppet, Chef)
> - Scripts personnalisés

---

## Constitution d'une GPO

> [!important] Les 3 composantes d'une GPO
> Une GPO est constituée de **trois composantes** distinctes qui travaillent ensemble :
> 
> 1. **Une entrée LDAP**
> 2. **Le contenu de la GPO**
> 3. **Un attribut gPLink**

---

## 1. L'entrée LDAP

> [!info] Emplacement et contenu
> L'**entrée LDAP GPO** est située sous :
> ```
> CN=Policies,CN=System,DC=domaine,DC=extension
> ```
> 
> Elle se trouve dans la **partition principale de l'Active Directory**.

### Informations contenues dans l'entrée LDAP

> [!note] Informations administratives
> L'entrée LDAP contient les **informations administratives** de la GPO :
> 
> - **Le nom** de la GPO (nom convivial)
> - **Le GUID** (identifiant unique)
> - **Les droits d'édition** de la GPO
> - **Délégation de droits** (qui peut modifier la GPO)
> 
> → Ce sont les **métadonnées** de la GPO

> [!example] Exemple de chemin LDAP d'une GPO
> ```
> CN={31B2F340-016D-11D2-945F-00C04FB984F9},CN=Policies,CN=System,DC=lab,DC=lan
> ```
> 
> Le GUID `{31B2F340-016D-11D2-945F-00C04FB984F9}` identifie de manière unique cette GPO.

---

## 2. Le contenu de la GPO

> [!info] Emplacement du contenu
> Le **contenu de la GPO** se trouve sur le serveur AD dans le partage **SYSVOL**.

### Structure du contenu

> [!note] Organisation des fichiers
> Le contenu est stocké dans un **répertoire dont le nom est le GUID** de la GPO :
> ```
> \\domaine\SYSVOL\domaine\Policies\{GUID}\
> ```
> 
> Ce répertoire contient **plusieurs fichiers d'instructions** qui définissent les actions de la GPO :
> - **GPT.INI** : Version de la GPO
> - **Machine/** : Paramètres ordinateur
> - **User/** : Paramètres utilisateur
> - Fichiers de configuration spécifiques

> [!example] Exemple de structure SYSVOL
> ```
> \\lab.lan\SYSVOL\lab.lan\Policies\{6AC1786C-016F-11D2-945F-00C04fB984F9}\
> ├── GPT.INI
> ├── Machine/
> │   ├── Registry.pol
> │   ├── Scripts/
> │   └── ...
> └── User/
>     ├── Registry.pol
>     ├── Scripts/
>     └── ...
> ```

> [!tip] Réplication via SYSVOL
> Le partage **SYSVOL** est **automatiquement répliqué** entre tous les contrôleurs de domaine, ce qui garantit que les GPO sont disponibles sur tous les DC.

---

## 3. L'attribut gPLink

> [!info] Liaison de la GPO
> L'attribut **gPLink** est affecté à une **OU** (Unité d'Organisation) ou à un **site AD**. Il établit le lien entre la GPO et l'emplacement où elle s'applique.

### Informations contenues dans gPLink

> [!note] Contenu de l'attribut gPLink
> L'attribut gPLink rassemble plusieurs informations :
> 
> - **L'identifiant de la GPO** (le GUID)
> - **Le chemin LDAP de la GPO**
> - **L'ordre de traitement** (priorité)
> - **L'application ou non** (activé/désactivé)

> [!example] Exemple d'attribut gPLink
> ```
> [LDAP://cn={31B2F340-016D-11D2-945F-00C04FB984F9},cn=policies,cn=system,DC=lab,DC=lan;0]
> ```
> 
> Le `0` à la fin indique l'état du lien (activé, désactivé, enforced).

---

## États des GPO

> [!abstract] Gestion de l'état des GPO
> En plus des caractéristiques vues précédemment, une GPO peut avoir **2 types d'états** qui contrôlent son application.

---

## État "Enforced" (Forcé)

> [!important] GPO Enforced
> Lorsqu'une GPO est marquée comme **"Enforced"** (forcée), elle a la **priorité sur les GPO appliquées à des niveaux inférieurs** dans la hiérarchie AD.

### Comportement d'une GPO Enforced

> [!warning] Priorité absolue
> Une GPO Enforced :
> - **Ignore le blocage d'héritage** des OU
> - **Écrase les paramètres** des GPO de niveau inférieur
> - **Doit être utilisée très rarement**
> - Est **typiquement réservée au Tier 0** (sécurité critique)

> [!example] Exemple d'utilisation
> **Scénario** :
> - Une GPO appliquée au **domaine** définit : "Mot de passe minimum 12 caractères"
> - Cette GPO est marquée **Enforced**
> - Une GPO appliquée à une **OU** définit : "Mot de passe minimum 8 caractères"
> 
> **Résultat** : Les objets de l'OU auront un mot de passe minimum de **12 caractères** (la GPO domaine Enforced prévaut).

> [!tip] Cas d'usage typique
> Utilise **Enforced** uniquement pour :
> - Politiques de sécurité **critiques** à l'échelle du domaine
> - Paramètres qui ne doivent **jamais** être contournés
> - GPO **Tier 0** (protection des contrôleurs de domaine)

---

## État "Enabled" / "Disabled" (Activé / Désactivé)

> [!info] État actif de la GPO
> L'état **"Enabled"** ou **"Disabled"** d'une GPO détermine si elle est **active ou non**.

### Différence : Link Enabled vs GPO Status Enabled

> [!important] Deux niveaux d'activation
> Il existe **deux concepts distincts** à ne pas confondre :

**1. Link Enabled (Lien activé)** :
> [!note] Activation du lien GPO-OU
> - Cette option détermine si le **lien entre une GPO et une OU** est actif
> - Si "Link Enabled" est **désactivé** → la GPO ne s'appliquera **pas aux objets dans cette OU**, même si la GPO elle-même est activée
> - **Contrôle** : Si le lien entre la GPO et l'OU est actif ou non
> - **Portée** : Spécifique à un lien particulier GPO-OU

**2. GPO Status Enabled (État de la GPO)** :
> [!note] Activation de la GPO elle-même
> - État de la **GPO elle-même**
> - Si elle est **désactivée** (Disabled), elle ne s'appliquera à **aucun objet**, indépendamment de l'état de ses liens
> - Une GPO doit être **activée** (Enabled) pour qu'elle puisse s'appliquer
> - **Contrôle** : Activation globale de la GPO
> - **Portée** : Tous les liens de la GPO

### Tableau comparatif

| Critère | Link Enabled | GPO Status Enabled |
|---------|--------------|-------------------|
| **Portée** | Un lien spécifique GPO-OU | La GPO entière |
| **Impact** | Désactive l'application sur une OU | Désactive l'application partout |
| **Utilisation** | Désactiver temporairement sur une OU | Désactiver complètement la GPO |

> [!example] Exemple de différence
> **Configuration** :
> - GPO "RestrictUSB" : Status = **Enabled**
> - Lien GPO → OU "Computers" : Link = **Enabled**
> - Lien GPO → OU "Servers" : Link = **Disabled**
> 
> **Résultat** :
> - Ordinateurs de l'OU "Computers" : GPO appliquée ✅
> - Ordinateurs de l'OU "Servers" : GPO non appliquée ❌

> [!tip] Quand utiliser chaque option ?
> **Désactiver un lien** (Link Disabled) :
> - Tester une GPO sur une OU spécifique
> - Exclure temporairement une OU d'une GPO
> 
> **Désactiver la GPO** (GPO Disabled) :
> - Mettre en maintenance une GPO
> - Désactiver complètement avant suppression

---

## Règles de priorité

> [!abstract] Ordre d'application des GPO
> Comprendre l'ordre dans lequel les GPO sont appliquées est crucial pour maîtriser leur comportement et résoudre les conflits de configuration.

---

## Stratégies locales

> [!quote] Définition
> Les **stratégies locales** sont un ensemble de configurations de sécurité et de gestion appliquées **directement à un ordinateur individuel**, sur les OS Microsoft.

### Caractéristiques

> [!info] Propriétés des stratégies locales
> Les stratégies locales :
> - Sont **définies localement** sur chaque machine
> - Ne dépendent **pas d'une gestion centralisée** par Active Directory
> - Fonctionnent en **Workgroup** (sans domaine) et en **domaine**
> - Sont éditées via la console locale **`gpedit.msc`**
> - S'appliquent même si l'ordinateur n'est pas membre d'un domaine

> [!note] Console d'édition
> Sur un poste local, utilise la commande :
> ```
> gpedit.msc
> ```
> 
> Pour éditer les stratégies locales de la machine.

---

## Stratégies locales vs GPO

> [!important] Comparaison des deux approches

| Critère | Stratégies locales | GPO (Active Directory) |
|---------|-------------------|------------------------|
| **Portée** | Spécifique à chaque ordinateur individuel | Centralisées via l'AD |
| **Environnement** | Workgroup (sans domaine) ET domaine | Uniquement en domaine |
| **Console** | Console locale : `gpedit.msc` | Console serveur : `gpmc.msc` |
| **Gestion** | Manuelle sur chaque poste | Centralisée depuis le serveur |
| **Priorité** | Appliquées en premier | **Priorité sur les stratégies locales** |
| **Réplication** | Aucune | Automatique entre DC |

> [!success] Avantage des GPO
> Les **GPO domaine** ont **toujours priorité** sur les stratégies locales. Cela garantit que les politiques centralisées prévalent sur les configurations locales.

> [!example] Exemple pratique
> **Situation** :
> - Stratégie locale sur PC1 : "Mot de passe minimum 6 caractères"
> - GPO du domaine : "Mot de passe minimum 10 caractères"
> - PC1 est membre du domaine
> 
> **Résultat** : La GPO du domaine s'applique → **10 caractères minimum**

---

## Priorité LSDOU

> [!important] Ordre d'application hiérarchique
> Lorsqu'un ordinateur dans un domaine AD démarre ou lorsqu'un utilisateur se connecte, les politiques sont appliquées dans l'ordre suivant :

### L'acronyme LSDOU

> [!note] Mémorisation : LSDOU
> **L** → **L**ocal  
> **S** → **S**ite  
> **D** → **D**omaine  
> **O** → **O**rganizational **U**nit (Unité d'Organisation)

### Ordre d'application détaillé

> [!info] Séquence d'application des GPO
> 
> **1. Local** :
> - D'abord, les **stratégies locales** de la machine sont appliquées
> - Définies via `gpedit.msc` sur le poste
> 
> **2. Site** :
> - Ensuite, les **GPO associées au site AD** sont appliquées
> - Les sites représentent des emplacements physiques (LAN)
> 
> **3. Domaine** :
> - Les **GPO de niveau domaine** sont appliquées après celles du site
> - S'appliquent à tous les objets du domaine
> 
> **4. OU (Unité d'Organisation)** :
> - Les **GPO des OU** sont appliquées en dernier
> - En commençant par **l'OU parent la plus élevée**
> - Et en **descendant** jusqu'à l'OU la plus spécifique
> 
> **⇒ Concept d'héritage hiérarchique**

### Principe de priorité

> [!success] Règle de priorité
> **La dernière GPO appliquée gagne** (principe LIFO - Last In, First Out pour les paramètres en conflit).
> 
> Si plusieurs GPO définissent le même paramètre avec des valeurs différentes :
> - La GPO appliquée **en dernier** (la plus spécifique - OU) **prévaut**
> - Sauf si une GPO de niveau supérieur est marquée **Enforced**

> [!example] Exemple de hiérarchie
> ```
> 1. Local (machine)
>    ↓
> 2. Site AD "Paris"
>    ↓
> 3. Domaine "lab.lan"
>    ↓
> 4. OU "France"
>    ↓
> 5. OU "Paris" (enfant de "France")
>    ↓
> 6. OU "IT" (enfant de "Paris")
> 
> → L'OU "IT" a la priorité la plus élevée
> ```

> [!tip] Mnémotechnique
> Pour retenir l'ordre **LSDOU**, pense à :
> **"Les Stratégies De l'Organisation l'Emportent"**
> 
> Ou visualise une pyramide inversée :
> ```
>   Local (base large - priorité faible)
>    Site
>   Domaine
>     OU (sommet - priorité forte)
> ```

---

## Héritage des GPO

> [!abstract] Transmission des GPO dans la hiérarchie
> L'héritage est le mécanisme par lequel les GPO des niveaux supérieurs s'appliquent aux niveaux inférieurs, sauf configuration contraire.

### Principe de l'héritage

> [!info] Propagation des GPO
> Par défaut, une GPO appliquée à un niveau parent est **héritée** par tous les niveaux enfants :
> - Une GPO liée au **domaine** s'applique à **toutes les OU**
> - Une GPO liée à une **OU parent** s'applique à **toutes les OU enfants**

### Blocage d'héritage

> [!warning] Block Inheritance
> Les **OU** peuvent être configurées pour **bloquer l'héritage** des GPO des niveaux supérieurs.
> 
> **Effet** :
> - Les GPO du domaine et des OU parents ne s'appliquent **plus**
> - Seules les GPO liées **directement à cette OU** s'appliquent
> - Les stratégies locales et de site s'appliquent toujours

> [!example] Exemple de blocage d'héritage
> ```
> Domaine "lab.lan"
> └─ GPO "Security-Global" (ex: Complexité mot de passe)
>    │
>    └─ OU "Developers"
>       └─ [Block Inheritance activé]
>          └─ GPO "Dev-Policy" (ex: Pas de complexité)
> 
> → Les développeurs n'héritent PAS de "Security-Global"
> → Seule "Dev-Policy" s'applique
> ```

### GPO Enforced (priorité absolue)

> [!important] Exception au blocage d'héritage
> Une GPO marquée comme **"Enforced"** :
> - **Écrasera** les politiques des niveaux inférieurs
> - S'appliquera **même si l'héritage est bloqué**
> - A une **priorité absolue**

> [!example] Exemple de GPO Enforced vs Block Inheritance
> ```
> Domaine "lab.lan"
> └─ GPO "Critical-Security" [ENFORCED]
>    │
>    └─ OU "Developers"
>       └─ [Block Inheritance activé]
>          └─ GPO "Dev-Policy"
> 
> → "Critical-Security" s'applique QUAND MÊME (Enforced)
> → "Dev-Policy" s'applique aussi
> → Ordre : Critical-Security, puis Dev-Policy
> ```

> [!tip] Quand utiliser le blocage d'héritage ?
> **Utilise le blocage d'héritage avec précaution** :
> - ✅ Pour des OU avec besoins **très spécifiques** (ex: serveurs de tests)
> - ✅ Pour des OU gérées par des **équipes différentes**
> - ❌ Évite d'en abuser → complexifie le dépannage
> - ❌ Les GPO Enforced contournent le blocage

---

## Fonctionnement sur une OU

> [!abstract] Priorité des GPO au sein d'une même OU
> Lorsque **plusieurs GPO** sont liées à une **même OU**, un ordre de traitement spécifique s'applique.

### Ordre de priorité sur une OU

> [!info] Trois facteurs déterminent la priorité
> 
> **1. Ordre déterminé manuellement** :
> - On peut spécifier **manuellement** un ordre de priorité
> - Via l'interface GPMC (Group Policy Management Console)
> - Ordre numérique : 1, 2, 3, etc.
> 
> **2. LIFO (Last In, First Out)** :
> - La **dernière GPO liée** est traitée **en premier**
> - Si 2 GPO ont des paramètres qui se chevauchent, le paramètre de la **dernière GPO traitée** prévaudra
> 
> **3. Filtrage et Sécurité** :
> - Les **paramètres de sécurité** et le **filtrage** (sécurité ou WMI) influencent l'application d'une GPO
> - Une GPO peut être filtrée pour ne s'appliquer qu'à certains utilisateurs/ordinateurs

> [!example] Exemple d'ordre sur une OU
> **Configuration sur OU "IT"** :
> ```
> GPO liées (ordre affiché) :
> 1. GPO-Wallpaper (Ordre: 3)
> 2. GPO-Security (Ordre: 2)
> 3. GPO-Software (Ordre: 1)
> ```
> 
> **Ordre de traitement** (du plus faible au plus fort) :
> ```
> 1. GPO-Wallpaper (traité en 1er - priorité faible)
> 2. GPO-Security (traité en 2ème)
> 3. GPO-Software (traité en dernier - priorité forte)
> ```
> 
> → Si GPO-Security définit "Bureau = bleu" et GPO-Software définit "Bureau = rouge"  
> → **Résultat** : Bureau rouge (GPO-Software traité en dernier)

### Ordre d'affichage vs Ordre d'application

> [!warning] Attention à la confusion
> Dans la console **GPMC** :
> - L'**ordre affiché** commence à **1** (en haut)
> - L'**ordre d'application** est **inverse** : la GPO en **bas** (dernier ordre) est appliquée **en dernier** et a donc la **priorité la plus forte**

> [!tip] Changer l'ordre des GPO
> Dans **GPMC** :
> 1. Sélectionne l'OU
> 2. Onglet "**Liaison de stratégie de groupe**"
> 3. Utilise les flèches **↑↓** pour changer l'ordre
> 4. GPO en **bas de la liste** = **priorité la plus forte**

---

## Filtrage de sécurité

> [!quote] Définition
> Le **filtrage de sécurité** permet de contrôler **à qui** (utilisateurs/ordinateurs) une GPO s'applique, en se basant sur les **permissions** sur l'objet GPO.

### Condition d'application d'une GPO

> [!important] Droits nécessaires
> Une GPO ne s'applique **que si** l'objet AD (utilisateur ou ordinateur) possède **DEUX droits** :
> 
> 1. **Read** (Lecture)
> 2. **Apply Group Policy** (Appliquer la stratégie de groupe)
> 
> Ces droits sont gérés via le **filtrage de sécurité** dans GPMC.

### Groupe par défaut : Authenticated Users

> [!info] Configuration par défaut
> Par défaut, le groupe **Authenticated Users** a le droit **Apply Group Policy** sur toutes les nouvelles GPO.
> 
> **Conséquence** :
> - **Tous les utilisateurs authentifiés** reçoivent la GPO
> - **Tous les ordinateurs authentifiés** reçoivent la GPO
> 
> → La GPO s'applique à **tout le monde** dans le périmètre de liaison (OU, domaine, site)

### Restreindre l'application d'une GPO

> [!success] Méthode de filtrage
> Pour **restreindre** une GPO à un sous-ensemble d'utilisateurs/ordinateurs :
> 
> **Étape 1** : Retirer l'application du groupe par défaut
> - Retirer **Authenticated Users** du filtrage de sécurité
> 
> **Étape 2** : Ajouter un groupe de sécurité AD
> - Ajouter un **groupe de sécurité AD** (Utilisateur ou Ordinateur)
> 
> **Étape 3** : Attribuer les droits
> - Lui donner le droit **Lecture** (Read)
> - Lui donner le droit **Apply Group Policy**
> 
> → La GPO s'appliquera **uniquement aux membres** de ce groupe

> [!example] Exemple de filtrage
> **Objectif** : Appliquer une GPO "Disable-USB" uniquement aux ordinateurs du groupe "Computers-Restricted"
> 
> **Configuration** :
> 1. Créer la GPO "Disable-USB"
> 2. Lier la GPO à l'OU "Computers"
> 3. **Filtrage de sécurité** :
>    - Retirer : **Authenticated Users**
>    - Ajouter : **Computers-Restricted** (groupe d'ordinateurs)
>    - Droits : **Read** + **Apply Group Policy**
> 
> **Résultat** :
> - Seuls les ordinateurs membres de "Computers-Restricted" reçoivent la GPO
> - Les autres ordinateurs de l'OU "Computers" ne reçoivent PAS la GPO

### Exclusion d'une GPO

> [!tip] Empêcher l'application d'une GPO
> Pour **exclure** certains utilisateurs/ordinateurs d'une GPO :
> 
> **Méthode** :
> - Ajouter le groupe au filtrage de sécurité
> - Attribuer le droit **Read** (Lecture)
> - Attribuer le droit **Deny - Apply Group Policy** (Refuser l'application)
> 
> → Les membres de ce groupe ne recevront **pas** la GPO, même s'ils sont dans le périmètre de liaison

> [!warning] Attention avec "Deny"
> L'utilisation de **Deny** est **puissante** et peut avoir des effets inattendus :
> - **Deny prévaut toujours** sur Allow
> - Peut bloquer l'application même si l'utilisateur est dans un autre groupe avec Allow
> - Privilégie le filtrage **positif** (retirer Authenticated Users + ajouter groupes spécifiques) plutôt que le filtrage **négatif** (Deny)

---

## Bonnes pratiques

> [!success] Recommandations pour une gestion optimale des GPO
> Voici les bonnes pratiques essentielles à suivre pour gérer efficacement les GPO dans ton infrastructure Active Directory.

### 1. Ne pas modifier les GPO par défaut

> [!warning] Default Domain Policy
> **Ne jamais modifier** les GPO de domaine par défaut :
> - **Default Domain Policy**
> - **Default Domain Controllers Policy**
> 
> **Raisons** :
> - Modifications difficiles à tracer
> - Risque de casser des fonctionnalités essentielles
> - Difficile de revenir en arrière
> 
> **Alternative** : Créer de **nouvelles GPO** et les lier au domaine ou aux OU

### 2. Se baser sur une hiérarchie d'OU

> [!tip] Structure organisée
> Organise ton Active Directory avec une **hiérarchie d'OU claire** :
> - Par **fonction** (IT, RH, Finance, etc.)
> - Par **localisation** (Paris, Lyon, Nantes, etc.)
> - Par **type de ressource** (Serveurs, Postes de travail, etc.)
> 
> → Facilite l'application ciblée des GPO

### 3. Nomenclature descriptive

> [!tip] Nommage clair
> Utilise une **nomenclature descriptive** pour tes GPO :
> 
> **Exemples de bonne nomenclature** :
> - `GPO-Security-PasswordPolicy`
> - `GPO-Computers-DisableUSB`
> - `GPO-Users-MapDrives`
> - `GPO-Servers-WindowsUpdate`
> 
> **Éviter** :
> - `GPO1`, `GPO2`, `Test`, `New GPO`

### 4. Ne pas utiliser les dossiers par défaut

> [!warning] Éviter Users et Computers
> **Ne pas utiliser** les conteneurs de base « **Users** » et « **Computers** » :
> - Ce sont des **containers**, pas des OU
> - On **ne peut pas lier de GPO** directement dessus
> - Pas de gestion hiérarchique
> 
> **Solution** : Créer des **OU dédiées** et y déplacer les objets

### 5. Supprimer un lien plutôt que désactiver

> [!tip] Gestion des liens
> Pour retirer une GPO d'une OU :
> - **Préférer** : **Supprimer le lien** de la GPO
> - **Éviter** : Désactiver le lien (laisse un lien inactif)
> 
> → Simplifie la compréhension et évite la confusion

### 6. Ne pas bloquer l'héritage

> [!warning] Block Inheritance
> **Utilise le blocage d'héritage avec parcimonie** :
> - Complexifie le dépannage
> - Rend la structure difficile à comprendre
> - Les GPO Enforced contournent le blocage
> 
> **Alternative** : Créer des GPO spécifiques pour les OU problématiques

### 7. Utiliser de petites GPO

> [!tip] GPO modulaires
> Crée des **GPO petites et ciblées** :
> - Une GPO = une fonction/un objectif
> - Plus facile à gérer et à dépanner
> - Meilleure réutilisabilité
> 
> **Exemple** :
> - ✅ `GPO-DisableUSB` + `GPO-MapDrives` + `GPO-Wallpaper`
> - ❌ `GPO-AllSettings` (tout dans une seule GPO)

### 8. Utiliser les gestions avancées de mot de passe

> [!tip] Fine-Grained Password Policy
> Utilise les **stratégies de mots de passe affinées** (FGPP) :
> - Permet d'avoir **différentes politiques de mots de passe** selon les groupes
> - Disponible depuis Windows Server 2008
> - Plus flexible que la politique de domaine unique
> 
> **Cas d'usage** :
> - Administrateurs : Politique stricte (16 caractères, 60 jours)
> - Utilisateurs standard : Politique normale (12 caractères, 90 jours)

### 9. Désactiver les configurations inutilisées

> [!tip] Optimisation des GPO
> **Désactive** les sections "Ordinateurs" ou "Utilisateurs" inutilisées dans une GPO :
> 
> - Si une GPO contient uniquement des paramètres **ordinateur** → Désactiver la section **Utilisateur**
> - Si une GPO contient uniquement des paramètres **utilisateur** → Désactiver la section **Ordinateur**
> 
> **Avantages** :
> - Réduit le temps de traitement des GPO
> - Améliore les performances de démarrage/connexion
> - Clarifie l'intention de la GPO

> [!example] Comment désactiver une section
> Dans GPMC :
> 1. Clic droit sur la GPO
> 2. **Propriétés** → Onglet **Détails**
> 3. **État de la GPO** :
>    - "Paramètres de l'ordinateur désactivés"
>    - "Paramètres de l'utilisateur désactivés"
>    - "Tous les paramètres désactivés"

### 10. Documenter les GPO

> [!tip] Documentation essentielle
> **Documente** chaque GPO :
> - **Objectif** de la GPO
> - **Paramètres** appliqués
> - **Raison** de la création
> - **Responsable** de la GPO
> - **Date** de création/modification
> 
> **Où documenter ?** :
> - Dans le champ **Commentaire** de la GPO (GPMC)
> - Dans une documentation centralisée (Wiki, SharePoint)

### Récapitulatif des bonnes pratiques

> [!success] Checklist des bonnes pratiques
> ✅ Ne pas modifier les GPO par défaut  
> ✅ Se baser sur une hiérarchie d'OU claire  
> ✅ Utiliser une nomenclature descriptive  
> ✅ Ne pas utiliser les conteneurs "Users" et "Computers"  
> ✅ Supprimer un lien plutôt que le désactiver  
> ✅ Limiter l'utilisation du blocage d'héritage  
> ✅ Créer de petites GPO modulaires  
> ✅ Utiliser les stratégies de mots de passe affinées  
> ✅ Désactiver les sections inutilisées  
> ✅ Documenter chaque GPO

---

## Points clés à retenir

> [!success] Synthèse pour le titre RNCP

### Définition et composants

**Qu'est-ce qu'une GPO ?**
- Collection virtuelle de politiques de sécurité
- Identifiée par un GUID unique
- Permet la gestion centralisée du parc informatique

**Les 3 composantes d'une GPO** :
1. **Entrée LDAP** : Métadonnées (nom, GUID, droits)
   - Emplacement : `CN=Policies,CN=System,DC=domaine,DC=ext`
2. **Contenu de la GPO** : Fichiers d'instructions
   - Emplacement : `\\domaine\SYSVOL\domaine\Policies\{GUID}\`
3. **Attribut gPLink** : Lien GPO-OU/Site
   - Contient : GUID, chemin LDAP, ordre, état

**Capacités de gestion** :
- Ordinateurs et utilisateurs
- Politiques de sécurité (restrictions, mots de passe)
- Interface graphique (bureau, menu démarrer)
- Déploiement (logiciels, scripts, services)
- Scripts de connexion
- Redirection de dossiers

### États des GPO

**Enforced (Forcé)** :
- Priorité absolue sur les niveaux inférieurs
- Ignore le blocage d'héritage
- À utiliser rarement (Tier 0)

**Enabled/Disabled** :
- **Link Enabled** : Active/désactive le lien GPO-OU
- **GPO Status Enabled** : Active/désactive la GPO entière

### Règles de priorité - LSDOU

> [!important] Ordre d'application
> **L** → **Local** (stratégies locales)  
> **S** → **Site** (GPO de site AD)  
> **D** → **Domain** (GPO de domaine)  
> **O** → **OU** (GPO d'Unité d'Organisation)
> 
> **Priorité** : Les GPO des **OU** ont la priorité la plus **forte** (appliquées en dernier)

**Héritage** :
- Par défaut, les GPO des niveaux supérieurs sont héritées
- **Block Inheritance** : Bloque l'héritage (sauf GPO Enforced)
- **GPO Enforced** : Contourne le blocage d'héritage

**Priorité sur une OU** :
- Ordre manuel configurable (1, 2, 3...)
- **LIFO** : Last In, First Out (dernière GPO liée = traité en premier)
- GPO en **bas de la liste** = priorité la plus **forte**

### Filtrage de sécurité

**Condition d'application** :
- Objet doit avoir 2 droits : **Read** + **Apply Group Policy**

**Groupe par défaut** :
- **Authenticated Users** → Tous les utilisateurs/ordinateurs authentifiés

**Restreindre une GPO** :
1. Retirer **Authenticated Users**
2. Ajouter un **groupe de sécurité** spécifique
3. Attribuer **Read** + **Apply Group Policy**

**Exclure de l'application** :
- Ajouter le groupe avec **Deny - Apply Group Policy**
- ⚠️ Utiliser avec précaution (Deny prévaut sur Allow)

### Bonnes pratiques

| Pratique | Recommandation |
|----------|---------------|
| **GPO par défaut** | ❌ Ne jamais modifier (Default Domain Policy) |
| **Structure** | ✅ Hiérarchie d'OU claire |
| **Nommage** | ✅ Nomenclature descriptive (GPO-Fonction-Action) |
| **Conteneurs** | ❌ Ne pas utiliser "Users" et "Computers" |
| **Liens** | ✅ Supprimer les liens plutôt que désactiver |
| **Héritage** | ⚠️ Limiter le blocage d'héritage |
| **Taille** | ✅ Petites GPO modulaires |
| **Mots de passe** | ✅ Utiliser FGPP (Fine-Grained Password Policy) |
| **Sections** | ✅ Désactiver les sections inutilisées |
| **Documentation** | ✅ Documenter chaque GPO |

### Commandes PowerShell essentielles

```powershell
# Lister toutes les GPO
Get-GPO -All

# Détails d'une GPO spécifique
Get-GPO -Name "GPO-Security"

# Rapport HTML d'une GPO
Get-GPOReport -Name "GPO-Security" -ReportType Html -Path "C:\report.html"

# Liens d'une GPO
Get-GPO -Name "GPO-Security" | Select DisplayName, GpoStatus, CreationTime

# Forcer la mise à jour des GPO sur un client
gpupdate /force

# Résultat des GPO appliquées (RSOP)
gpresult /r
gpresult /h report.html
```

### Dépannage GPO

**Problèmes courants** :
- GPO ne s'applique pas → Vérifier filtrage de sécurité
- Conflit entre GPO → Vérifier ordre LSDOU et Enforced
- Réplication lente → Vérifier SYSVOL et réplication DC

**Outils de diagnostic** :
- `gpupdate /force` : Forcer l'application
- `gpresult /r` : Voir les GPO appliquées
- **RSOP** (Resultant Set of Policy) : Simulation
- **GPMC** : Group Policy Management Console

---

## Glossaire technique

> [!note] Définitions essentielles pour le TSSR

| Terme | Définition |
|-------|------------|
| **GPO** | Group Policy Object - Collection de politiques de sécurité et de configuration |
| **GUID** | Globally Unique Identifier - Identifiant unique d'une GPO sur 128 bits |
| **SYSVOL** | Partage réseau répliqué entre DC contenant les fichiers de GPO et scripts |
| **gPLink** | Attribut LDAP liant une GPO à une OU/Site/Domaine |
| **LSDOU** | Ordre d'application : Local, Site, Domain, Organizational Unit |
| **Enforced** | État forçant l'application d'une GPO avec priorité absolue |
| **Block Inheritance** | Blocage de l'héritage des GPO des niveaux supérieurs |
| **Link Enabled** | État du lien entre une GPO et une OU (actif/inactif) |
| **GPO Status Enabled** | État global de la GPO (activée/désactivée) |
| **Filtrage de sécurité** | Contrôle de l'application d'une GPO via permissions Read et Apply Group Policy |
| **Authenticated Users** | Groupe par défaut ayant le droit d'appliquer les GPO |
| **LIFO** | Last In, First Out - Principe de priorité des GPO sur une même OU |
| **FGPP** | Fine-Grained Password Policy - Stratégies de mots de passe affinées par groupe |
| **GPMC** | Group Policy Management Console - Console de gestion des GPO (gpmc.msc) |
| **gpedit.msc** | Éditeur de stratégies de groupe locales |
| **gpupdate** | Commande forçant la mise à jour des GPO sur un client |
| **gpresult** | Commande affichant les GPO appliquées à un utilisateur/ordinateur |
| **RSOP** | Resultant Set of Policy - Ensemble résultant des stratégies appliquées |
| **Computer Configuration** | Section GPO avec paramètres appliqués aux ordinateurs |
| **User Configuration** | Section GPO avec paramètres appliqués aux utilisateurs |

---

## Conclusion

> [!success] Ce que tu dois maîtriser
> Après l'étude de ce cours sur les GPO, tu dois être capable de :
> 
> ✅ **Définir** ce qu'est une GPO et ses composants :
>    - Entrée LDAP, Contenu SYSVOL, Attribut gPLink
> 
> ✅ **Comprendre** les capacités de gestion des GPO :
>    - Ordinateurs, utilisateurs, sécurité, déploiement
> 
> ✅ **Maîtriser** les états des GPO :
>    - Enforced vs Enabled
>    - Link Enabled vs GPO Status Enabled
> 
> ✅ **Appliquer** les règles de priorité :
>    - LSDOU (Local, Site, Domain, OU)
>    - Héritage et blocage d'héritage
>    - Ordre de traitement sur une OU (LIFO)
> 
> ✅ **Configurer** le filtrage de sécurité :
>    - Read + Apply Group Policy
>    - Restreindre l'application aux groupes spécifiques
> 
> ✅ **Appliquer** les bonnes pratiques :
>    - Ne pas modifier les GPO par défaut
>    - Structure d'OU claire
>    - Nomenclature descriptive
>    - GPO modulaires
>    - Documentation

---

## Liens et ressources

> [!info] Aucun lien externe n'était présent dans le document PowerPoint source
> Ce document de révision a été créé exclusivement à partir du contenu des slides du cours.

---

**Document créé le** : Janvier 2026  
**Format** : Markdown pour Obsidian  
**Version** : 1.0  
**Auteur** : Support de révision TSSR
