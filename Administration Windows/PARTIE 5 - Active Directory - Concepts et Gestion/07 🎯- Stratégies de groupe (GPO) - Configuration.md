## 📋 Table des matières

- [[#🎯 Stratégies de groupe (GPO) - Configuration|Introduction]]
- [[#📊 Configuration ordinateur vs Configuration utilisateur|Configuration ordinateur vs utilisateur]]
  - [[#Différences fondamentales|Différences fondamentales]]
  - [[#Ordre de traitement et priorité|Ordre de traitement et priorité]]
  - [[#Quand utiliser chaque type|Quand utiliser chaque type]]
- [[#🔒 Paramètres de sécurité via GPO|Paramètres de sécurité via GPO]]
  - [[#Stratégies de mot de passe|Stratégies de mot de passe]]
  - [[#Stratégies de verrouillage de compte|Stratégies de verrouillage de compte]]
  - [[#Stratégies d'audit|Stratégies d'audit]]
  - [[#Restriction logicielle|Restriction logicielle]]
  - [[#Pare-feu Windows|Pare-feu Windows]]
- [[#📁 Redirection de dossiers|Redirection de dossiers]]
  - [[#Pourquoi rediriger les dossiers|Pourquoi rediriger les dossiers]]
  - [[#Dossiers pouvant être redirigés|Dossiers pouvant être redirigés]]
  - [[#Configuration de la redirection|Configuration de la redirection]]
  - [[#Gestion des autorisations|Gestion des autorisations]]
- [[#⚙️ Scripts de démarrage, arrêt, ouverture et fermeture de session|Scripts]]
  - [[#Scripts de démarrage ordinateur|Scripts de démarrage ordinateur]]
  - [[#Scripts d'arrêt ordinateur|Scripts d'arrêt ordinateur]]
  - [[#Scripts d'ouverture de session|Scripts d'ouverture de session]]
  - [[#Scripts de fermeture de session|Scripts de fermeture de session]]
  - [[#Bonnes pratiques pour les scripts|Bonnes pratiques pour les scripts]]
- [[#🎨 Préférences de stratégie de groupe|Préférences de stratégie de groupe]]
  - [[#Différence entre Stratégies et Préférences|Différence entre Stratégies et Préférences]]
  - [[#Types de préférences disponibles|Types de préférences disponibles]]
  - [[#Ciblage des préférences|Ciblage des préférences]]
- [[#📦 Installation de logiciels via GPO|Installation de logiciels via GPO]]
  - [[#Méthodes de déploiement|Méthodes de déploiement]]
  - [[#Packages MSI et transformation|Packages MSI et transformation]]
  - [[#Publication vs Attribution|Publication vs Attribution]]
  - [[#Gestion des mises à jour et désinstallations|Gestion des mises à jour et désinstallations]]

---

## Introduction

> [!abstract] Vue d'ensemble
> Les Stratégies de groupe (GPO - Group Policy Objects) sont l'un des outils les plus puissants d'Active Directory pour la gestion centralisée de la configuration des ordinateurs et des utilisateurs. Ce cours approfondit les différents aspects de la configuration des GPO.

> [!important] Pourquoi maîtriser la configuration des GPO ?
> - **Cohérence** : Assure une configuration uniforme sur l'ensemble du parc informatique
> - **Sécurité** : Applique des politiques de sécurité strictes et cohérentes
> - **Automatisation** : Réduit considérablement le travail manuel d'administration
> - **Conformité** : Garantit le respect des normes et réglementations
> - **Efficacité** : Centralise la gestion de milliers de machines et d'utilisateurs

---

## 📊 Configuration ordinateur vs Configuration utilisateur

> [!info] Structure duale des GPO
> Chaque GPO est divisée en deux grandes sections principales, chacune ayant un rôle et un moment d'application spécifiques.

### Différences fondamentales

| Critère | Configuration ordinateur | Configuration utilisateur |
|---------|-------------------------|---------------------------|
| **Application** | Au démarrage de l'ordinateur | À l'ouverture de session utilisateur |
| **Cible** | L'ordinateur, quelle que soit la personne qui l'utilise | L'utilisateur, quel que soit l'ordinateur utilisé |
| **Contexte** | Avant l'ouverture de session | Après l'ouverture de session |
| **Privilèges** | Système (élevés) | Utilisateur (limités) |
| **Rafraîchissement** | Toutes les 90-120 min (ordinateur allumé) | Toutes les 90-120 min (session ouverte) |
| **Force en cas de conflit** | **Prioritaire** (écrase la config utilisateur) | Moins prioritaire |

> [!example] Exemple de distinction
> **Configuration ordinateur** :
> - Installation d'un antivirus sur tous les postes
> - Configuration du pare-feu Windows
> - Paramètres de sécurité locale
> - Scripts de démarrage système
> 
> **Configuration utilisateur** :
> - Mappage de lecteurs réseau personnalisés
> - Fond d'écran et thème du bureau
> - Redirection du dossier Documents
> - Scripts d'ouverture de session

### Ordre de traitement et priorité

```
1. DÉMARRAGE DE L'ORDINATEUR
   └─> Application de la Configuration ordinateur
       ├─ GPO local (ordinateur)
       ├─ GPO du site
       ├─ GPO du domaine
       └─ GPO des OU (du plus haut au plus bas)

2. OUVERTURE DE SESSION UTILISATEUR
   └─> Application de la Configuration utilisateur
       ├─ GPO local (utilisateur)
       ├─ GPO du site
       ├─ GPO du domaine
       └─ GPO des OU (du plus haut au plus bas)
```

> [!warning] Conflit entre configuration ordinateur et utilisateur
> Lorsqu'un même paramètre est configuré dans les deux sections, la **Configuration ordinateur l'emporte TOUJOURS** sur la Configuration utilisateur.
> 
> Exemple : Si la config ordinateur désactive le Panneau de configuration et que la config utilisateur l'active, le Panneau de configuration sera **désactivé**.

### Quand utiliser chaque type

#### Utiliser la Configuration ordinateur pour :

> [!tip] Configuration ordinateur
> - Tout ce qui concerne la **sécurité système**
> - Les **installations de logiciels** qui nécessitent des droits administrateur
> - Les paramètres qui doivent s'appliquer **quel que soit l'utilisateur**
> - La configuration **matérielle et réseau**
> - Les paramètres du **système d'exploitation**

```
Exemples concrets :
✅ Désactiver USB
✅ Configurer BitLocker
✅ Installer des imprimantes réseau
✅ Paramètres IPsec
✅ Configuration du pare-feu
✅ Stratégies d'audit de sécurité
```

#### Utiliser la Configuration utilisateur pour :

> [!tip] Configuration utilisateur
> - Les paramètres **personnalisés par utilisateur**
> - Ce qui doit **suivre l'utilisateur** d'un ordinateur à l'autre
> - Les préférences **d'interface utilisateur**
> - Les **mappages réseau** personnalisés
> - La **redirection de dossiers** utilisateur

```
Exemples concrets :
✅ Redirection Documents, Bureau, Favoris
✅ Lecteurs réseau mappés (H:, P:)
✅ Paramètres Internet Explorer/Edge
✅ Fond d'écran et écran de verrouillage
✅ Thème Windows
✅ Raccourcis personnalisés
```

> [!warning] Piège courant
> Ne pas configurer le même paramètre dans les deux sections avec des valeurs différentes, sauf si vous comprenez parfaitement la priorité. Cela crée de la confusion et des comportements inattendus.

---

## 🔒 Paramètres de sécurité via GPO

> [!abstract] Centralisation de la sécurité
> Les GPO permettent d'appliquer des paramètres de sécurité uniformes sur l'ensemble du domaine, garantissant conformité et protection.

### Stratégies de mot de passe

> [!info] Emplacement
> **Configuration ordinateur** → Stratégies → Paramètres Windows → Paramètres de sécurité → Stratégies de compte → Stratégie de mot de passe

#### Paramètres essentiels

| Paramètre | Description | Recommandation ANSSI |
|-----------|-------------|----------------------|
| **Longueur minimale** | Nombre minimum de caractères | Minimum 12 caractères |
| **Complexité** | Exige majuscules, minuscules, chiffres, caractères spéciaux | Activée |
| **Âge maximal** | Durée de validité du mot de passe | 90 jours maximum |
| **Âge minimal** | Délai avant de pouvoir changer à nouveau | 1 jour (empêche le recyclage immédiat) |
| **Historique** | Nombre de mots de passe mémorisés | 24 derniers mots de passe |
| **Stockage réversible** | Chiffrement réversible des mots de passe | **Désactivé** (sauf besoin spécifique) |

> [!example] Configuration recommandée pour une PME
> ```
> ✅ Longueur minimale : 12 caractères
> ✅ Complexité requise : Activée
> ✅ Âge maximal : 90 jours
> ✅ Âge minimal : 1 jour
> ✅ Historique : 24 mots de passe
> ❌ Stockage réversible : Désactivé
> ```

> [!warning] Attention aux stratégies de mots de passe
> - Les stratégies de mot de passe s'appliquent **au niveau du domaine**
> - Impossible d'avoir des stratégies différentes par OU (sauf avec les Fine-Grained Password Policies - PSO)
> - La GPO doit être liée au **domaine**, pas à une OU

### Stratégies de verrouillage de compte

> [!info] Emplacement
> **Configuration ordinateur** → Stratégies → Paramètres Windows → Paramètres de sécurité → Stratégies de compte → Stratégie de verrouillage de compte

#### Protection contre les attaques par force brute

| Paramètre | Description | Recommandation |
|-----------|-------------|----------------|
| **Seuil de verrouillage** | Nombre de tentatives échouées avant verrouillage | 5 tentatives |
| **Durée de verrouillage** | Temps pendant lequel le compte reste verrouillé | 30 minutes |
| **Réinitialisation du compteur** | Délai avant remise à zéro du compteur de tentatives | 30 minutes |

> [!tip] Astuce sécurité
> Activer le verrouillage de compte protège contre les attaques par force brute, mais attention aux attaques par déni de service (DoS) : un attaquant pourrait verrouiller de nombreux comptes volontairement.
> 
> **Solution** : Surveiller les événements de verrouillage (Event ID 4740) et utiliser une durée de verrouillage raisonnable.

### Stratégies d'audit

> [!info] Emplacement
> **Configuration ordinateur** → Stratégies → Paramètres Windows → Paramètres de sécurité → Stratégies locales → Stratégie d'audit

Les stratégies d'audit permettent d'enregistrer des événements de sécurité dans les journaux Windows.

#### Catégories d'audit à activer

| Catégorie | Événements enregistrés | Recommandation |
|-----------|------------------------|----------------|
| **Ouverture de session** | Connexions et déconnexions | Réussite et Échec |
| **Gestion des comptes** | Création, modification, suppression de comptes | Réussite et Échec |
| **Accès aux objets** | Accès aux fichiers, dossiers, registre | Échec (minimum) |
| **Modification de stratégie** | Changements de stratégies de sécurité | Réussite et Échec |
| **Utilisation de privilèges** | Utilisation de droits administratifs | Réussite et Échec |
| **Suivi des processus** | Démarrage et arrêt de programmes | Selon besoin |
| **Événements système** | Démarrage, arrêt, redémarrage | Réussite et Échec |

> [!example] Configuration d'audit recommandée
> ```
> Audit de l'ouverture de session
>   ✅ Réussite : OUI (traçabilité)
>   ✅ Échec : OUI (tentatives d'intrusion)
> 
> Audit de la gestion des comptes
>   ✅ Réussite : OUI (qui crée/modifie des comptes)
>   ✅ Échec : OUI (tentatives non autorisées)
> 
> Audit d'accès aux objets
>   ⚠️  Réussite : NON (trop verbeux)
>   ✅ Échec : OUI (tentatives d'accès non autorisées)
> 
> Audit de modification de stratégie
>   ✅ Réussite : OUI (traçabilité des changements)
>   ✅ Échec : OUI (tentatives non autorisées)
> ```

> [!warning] Gestion de l'espace disque
> L'audit génère un grand volume de journaux. Configurer :
> - La **taille maximale des journaux** (Événements de sécurité : minimum 1 Go)
> - La **stratégie de rétention** (écraser si nécessaire ou archiver)
> - Une solution de **centralisation des logs** (SIEM) pour l'analyse

### Restriction logicielle

> [!info] Emplacement
> **Configuration ordinateur** → Stratégies → Paramètres Windows → Paramètres de sécurité → Stratégies de restriction logicielle

> [!tip] Alternative moderne : AppLocker
> Les stratégies de restriction logicielle sont l'ancienne méthode. **AppLocker** (disponible dans les éditions Enterprise/Education) offre plus de fonctionnalités et de flexibilité.

#### Modes de restriction

| Mode | Description | Usage |
|------|-------------|-------|
| **Non restreint** | Tout peut s'exécuter par défaut | Ajouter des règles de blocage spécifiques |
| **Interdit** | Rien ne peut s'exécuter par défaut | Ajouter des règles d'autorisation (whitelist) |

> [!example] Exemple de restriction - Bloquer l'exécution depuis les profils temporaires
> ```
> Objectif : Empêcher l'exécution de programmes depuis %TEMP% et %TMP%
> 
> Chemin d'accès à bloquer :
> - %LocalAppData%\Temp
> - %Temp%
> - %TMP%
> 
> Niveau de sécurité : Interdit
> Type de règle : Règle de chemin d'accès
> ```

> [!warning] Attention au mode "Interdit"
> En mode "Interdit" par défaut, vous devez **explicitement autoriser** tous les chemins légitimes :
> - C:\Windows
> - C:\Program Files
> - C:\Program Files (x86)
> 
> Sinon, Windows ne pourra plus démarrer correctement !

### Pare-feu Windows

> [!info] Emplacement
> **Configuration ordinateur** → Stratégies → Paramètres Windows → Paramètres de sécurité → Pare-feu Windows Defender avec fonctions avancées de sécurité

#### Profils de pare-feu

Le pare-feu Windows utilise trois profils selon l'emplacement réseau :

| Profil | Quand il s'applique | Niveau de restriction recommandé |
|--------|---------------------|----------------------------------|
| **Domaine** | Ordinateur connecté au domaine AD | Modéré (confiance réseau interne) |
| **Privé** | Réseau domestique ou de confiance | Modéré à élevé |
| **Public** | Réseau public (Wi-Fi, hôtel, aéroport) | **Élevé** (méfiance par défaut) |

> [!example] Configuration pare-feu via GPO
> ```
> Profil Domaine :
>   ✅ Pare-feu : Activé
>   ✅ Connexions entrantes : Bloquer (par défaut)
>   ✅ Connexions sortantes : Autoriser
>   ✅ Notifications : Désactivées (domaine géré)
> 
> Profil Public :
>   ✅ Pare-feu : Activé
>   ✅ Connexions entrantes : Bloquer
>   ✅ Connexions sortantes : Autoriser
>   ✅ Notifications : Activées (alerter l'utilisateur)
> ```

#### Création de règles de pare-feu

> [!tip] Autoriser un service via GPO
> ```
> Exemple : Autoriser l'accès RDP (Remote Desktop) pour les admins
> 
> Type de règle : Prédéfinie (Bureau à distance)
> Action : Autoriser la connexion
> Profils : Domaine uniquement
> 
> OU créer une règle personnalisée :
> Protocole : TCP
> Port : 3389
> Adresses IP distantes : 192.168.1.0/24 (réseau admin)
> Action : Autoriser
> Profil : Domaine
> ```

> [!warning] Sécurité des règles de pare-feu
> - **Évitez les règles "Autoriser tout"**
> - Spécifiez toujours les **ports, protocoles et adresses IP** précis
> - Documentez chaque règle (nom descriptif)
> - Révisez régulièrement les règles actives

---

## 📁 Redirection de dossiers

> [!abstract] Mobilité des données utilisateur
> La redirection de dossiers permet de stocker les données utilisateur sur un serveur de fichiers plutôt que localement, offrant sauvegarde, mobilité et centralisation.

### Pourquoi rediriger les dossiers ?

> [!tip] Avantages de la redirection
> ✅ **Sauvegarde centralisée** : Les données sont sauvegardées avec le serveur
> ✅ **Mobilité utilisateur** : Les données suivent l'utilisateur d'un poste à l'autre
> ✅ **Économie d'espace local** : Les disques SSD des postes sont libérés
> ✅ **Accès distant** : Les fichiers sont accessibles via le réseau
> ✅ **Quotas centralisés** : Gestion de l'espace de stockage par utilisateur
> ✅ **Profils itinérants allégés** : Moins de données à synchroniser

> [!warning] Inconvénients et prérequis
> ⚠️ **Dépendance réseau** : Sans réseau, pas d'accès aux dossiers
> ⚠️ **Performances** : Peut être plus lent que l'accès local (selon la bande passante)
> ⚠️ **Disponibilité** : Si le serveur est hors ligne, les données sont inaccessibles
> ⚠️ **Nécessite** : Un serveur de fichiers fiable avec quotas et sauvegardes

### Dossiers pouvant être redirigés

> [!info] Emplacement
> **Configuration utilisateur** → Stratégies → Paramètres Windows → Redirection de dossiers

#### Dossiers couramment redirigés

| Dossier | Chemin local par défaut | Recommandation | Taille moyenne |
|---------|-------------------------|----------------|----------------|
| **Documents** | `%UserProfile%\Documents` | ✅ **Toujours rediriger** | Variable (1-10 Go) |
| **Bureau** | `%UserProfile%\Desktop` | ✅ Rediriger | 100-500 Mo |
| **Images** | `%UserProfile%\Pictures` | ⚠️ Selon usage | 1-20 Go |
| **Musique** | `%UserProfile%\Music` | ❌ Généralement non | 5-50 Go |
| **Vidéos** | `%UserProfile%\Videos` | ❌ Non (trop volumineux) | 10-100+ Go |
| **Favoris** | `%UserProfile%\Favorites` | ✅ Rediriger | < 10 Mo |
| **Menu Démarrer** | `%AppData%\Microsoft\Windows\Start Menu` | ⚠️ Selon besoin | < 50 Mo |
| **Téléchargements** | `%UserProfile%\Downloads` | ⚠️ Selon politique | Variable |
| **AppData\Roaming** | `%UserProfile%\AppData\Roaming` | ⚠️ Profils itinérants | 100 Mo - 2 Go |

> [!tip] Recommandation pour une PME
> ```
> ✅ Rediriger :
>    - Documents (priorité 1)
>    - Bureau (priorité 2)
>    - Favoris (priorité 3)
> 
> ⚠️ Évaluer selon usage :
>    - Images (si photographes/designers)
>    - Téléchargements (si besoin de nettoyage automatique)
> 
> ❌ Ne pas rediriger :
>    - Vidéos (trop volumineux)
>    - Musique (personnel, volumineux)
> ```

### Configuration de la redirection

> [!example] Configuration étape par étape - Redirection du dossier Documents

#### 1. Préparation du serveur de fichiers

```powershell
# Créer un partage pour les redirections de dossiers
# Sur le serveur de fichiers

# Créer le dossier racine
New-Item -Path "D:\Redirections" -ItemType Directory

# Créer un partage SMB
New-SmbShare -Name "Redirections$" -Path "D:\Redirections" -FullAccess "Admins du domaine" -ChangeAccess "Utilisateurs du domaine"

# Activer l'énumération basée sur l'accès (ABE)
Set-SmbShare -Name "Redirections$" -FolderEnumerationMode AccessBased
```

> [!info] Pourquoi le $ à la fin ?
> Le `$` rend le partage **caché**. Il n'apparaîtra pas dans la navigation réseau, mais reste accessible via `\\serveur\Redirections$`

#### 2. Configuration des autorisations NTFS

```
Structure recommandée :
\\Serveur\Redirections$
  ├─ %username%
  │   ├─ Documents
  │   ├─ Bureau
  │   └─ Favoris

Autorisations sur D:\Redirections :
✅ SYSTEM : Contrôle total
✅ Admins du domaine : Contrôle total
✅ Utilisateurs du domaine : 
   - Lister le dossier / Lire les données
   - Créer des dossiers / Ajouter des données
✅ CREATOR OWNER : Contrôle total (sous-dossiers et fichiers uniquement)
```

> [!warning] Autorisations critiques
> L'utilisateur ne doit **PAS** avoir "Contrôle total" sur la racine `D:\Redirections`, sinon il pourrait voir/modifier les dossiers des autres utilisateurs !

#### 3. Configuration dans la GPO

```
Emplacement GPO :
Configuration utilisateur
  └─ Stratégies
     └─ Paramètres Windows
        └─ Redirection de dossiers
           └─ Documents

Paramèttre : De base - Rediriger le dossier de tout le monde au même emplacement

Chemin cible : \\Serveur\Redirections$\%username%\Documents

Options :
✅ Accorder à l'utilisateur des droits exclusifs
✅ Déplacer le contenu vers le nouvel emplacement
✅ Appliquer également la stratégie de redirection aux systèmes d'exploitation Windows 2000, Windows 2000 Server, Windows XP et Windows Server 2003
```

> [!tip] Variable %username%
> Windows crée automatiquement un sous-dossier au nom de l'utilisateur connecté. Chaque utilisateur aura son propre dossier : `\\Serveur\Redirections$\jdupont\Documents`

#### 4. Options avancées

| Option | Description | Recommandation |
|--------|-------------|----------------|
| **Accorder droits exclusifs** | Seul l'utilisateur peut accéder à son dossier | ✅ Activé (confidentialité) |
| **Déplacer le contenu** | Copie les fichiers existants vers le nouveau chemin | ✅ Activé (première fois) |
| **Laisser le dossier dans le nouvel emplacement** | Conserve les fichiers sur le serveur même si la GPO est supprimée | ✅ Activé (sécurité des données) |
| **Rediriger vers le profil local** | Redirection vers le profil local si serveur inaccessible | ⚠️ Selon besoin |

### Gestion des autorisations

> [!example] Autorisations NTFS détaillées sur un dossier utilisateur

```
D:\Redirections\jdupont
│
Autorisations héritées de D:\Redirections :
✅ SYSTEM : Contrôle total
✅ Admins du domaine : Contrôle total

Autorisations explicites (ajoutées automatiquement par GPO) :
✅ jdupont : Contrôle total (ce dossier, les sous-dossiers et les fichiers)
❌ Autres utilisateurs : Aucun accès
```

> [!warning] Piège courant - Héritage des autorisations
> Si vous n'activez pas "Accorder à l'utilisateur des droits exclusifs" dans la GPO, Windows ne supprime pas l'héritage et tous les utilisateurs du domaine pourraient accéder aux dossiers des autres !

---

## ⚙️ Scripts de démarrage, arrêt, ouverture et fermeture de session

> [!abstract] Automatisation via scripts
> Les GPO permettent d'exécuter automatiquement des scripts à différents moments du cycle de vie de l'ordinateur et de la session utilisateur.

### Vue d'ensemble des 4 types de scripts

| Type | Moment d'exécution | Contexte | Configuration |
|------|-------------------|----------|---------------|
| **Démarrage ordinateur** | Au démarrage de l'ordinateur (avant l'ouverture de session) | SYSTEM (droits admin) | Configuration ordinateur |
| **Arrêt ordinateur** | À l'arrêt ou au redémarrage | SYSTEM (droits admin) | Configuration ordinateur |
| **Ouverture de session** | À l'ouverture de session utilisateur | Utilisateur (droits limités) | Configuration utilisateur |
| **Fermeture de session** | À la fermeture de session | Utilisateur (droits limités) | Configuration utilisateur |

### Scripts de démarrage ordinateur

> [!info] Emplacement
> **Configuration ordinateur** → Stratégies → Paramètres Windows → Scripts (démarrage/arrêt) → Démarrage

> [!tip] Cas d'usage typiques
> - **Synchronisation horaire** avec le serveur NTP
> - **Mappage de lecteurs réseau** système (partages applicatifs)
> - **Nettoyage de fichiers temporaires** système
> - **Configuration d'imprimantes** réseau par défaut
> - **Montage de volumes** ou connexion iSCSI
> - **Mises à jour de sécurité** ou de logiciels

> [!example] Script de démarrage - Nettoyage des fichiers temporaires
> ```batch
> @echo off
> REM Script de nettoyage au démarrage
> REM S'exécute avec les droits SYSTEM
> 
> REM Nettoyage du dossier Temp Windows
> del /q /f /s "%WINDIR%\Temp\*.*" >nul 2>&1
> 
> REM Nettoyage des fichiers temporaires Internet Explorer
> del /q /f /s "%WINDIR%\Temp\Temporary Internet Files\*.*" >nul 2>&1
> 
> REM Nettoyage du cache DNS
> ipconfig /flushdns >nul 2>&1
> 
> REM Journalisation
> echo %DATE% %TIME% - Nettoyage effectué >> C:\Windows\Logs\cleanup.log
> ```

> [!example] Script PowerShell - Synchronisation horaire
> ```powershell
> # Script de synchronisation horaire au démarrage
> # S'exécute avec les droits SYSTEM
> 
> # Configurer le serveur NTP du domaine
> $NtpServer = "dc01.entreprise.local"
> 
> # Configurer W32Time
> w32tm /config /manualpeerlist:$NtpServer /syncfromflags:manual /reliable:yes /update
> 
> # Resynchroniser immédiatement
> w32tm /resync /force
> 
> # Journaliser
> Add-Content -Path "C:\Windows\Logs\timesync.log" -Value "$(Get-Date) - Synchronisation avec $NtpServer effectuée"
> ```

### Scripts d'arrêt ordinateur

> [!info] Emplacement
> **Configuration ordinateur** → Stratégies → Paramètres Windows → Scripts (démarrage/arrêt) → Arrêt

> [!warning] Limitation de temps
> Par défaut, Windows attend **maximum 10 minutes** pour l'exécution des scripts d'arrêt. Au-delà, le système force l'arrêt.
> 
> Configurer via GPO si nécessaire :
> `Configuration ordinateur → Modèles d'administration → Système → Scripts → Durée maximale d'attente pour les scripts de stratégie de groupe`

> [!tip] Cas d'usage typiques
> - **Sauvegarde de fichiers** critiques avant arrêt
> - **Déconnexion de partages** ou volumes réseau
> - **Nettoyage de logs** locaux
> - **Envoi de statistiques** d'utilisation
> - **Archivage de données** locales

> [!example] Script d'arrêt - Sauvegarde des logs avant arrêt
> ```batch
> @echo off
> REM Script de sauvegarde des logs à l'arrêt
> REM S'exécute avec les droits SYSTEM
> 
> REM Définir le chemin de destination
> set BACKUP_PATH=\\Serveur\Sauvegardes$\Logs\%COMPUTERNAME%
> 
> REM Créer le dossier si inexistant
> if not exist "%BACKUP_PATH%" mkdir "%BACKUP_PATH%"
> 
> REM Copier les logs système importants
> xcopy "%WINDIR%\System32\winevt\Logs\System.evtx" "%BACKUP_PATH%\" /Y /Q
> xcopy "%WINDIR%\System32\winevt\Logs\Security.evtx" "%BACKUP_PATH%\" /Y /Q
> xcopy "%WINDIR%\System32\winevt\Logs\Application.evtx" "%BACKUP_PATH%\" /Y /Q
> 
> REM Horodater la sauvegarde
> echo Sauvegarde effectuée le %DATE% à %TIME% > "%BACKUP_PATH%\derniere_sauvegarde.txt"
> ```

### Scripts d'ouverture de session

> [!info] Emplacement
> **Configuration utilisateur** → Stratégies → Paramètres Windows → Scripts (ouverture/fermeture de session) → Ouverture de session

> [!important] Contexte utilisateur
> Ces scripts s'exécutent avec les **droits de l'utilisateur**, pas avec des droits administrateur. Ils ne peuvent donc pas effectuer d'opérations système nécessitant des privilèges élevés.

> [!tip] Cas d'usage typiques
> - **Mappage de lecteurs réseau** utilisateur
> - **Affichage de messages** d'information
> - **Synchronisation de fichiers** utilisateur
> - **Configuration de l'environnement** de travail
> - **Ouverture d'applications** au démarrage
> - **Mise à jour du bureau** et des raccourcis

> [!example] Script d'ouverture de session - Mappage de lecteurs réseau
> ```batch
> @echo off
> REM Script de mappage de lecteurs réseau
> REM S'exécute avec les droits de l'utilisateur connecté
> 
> REM Supprimer d'abord tous les lecteurs mappés (nettoyage)
> net use * /delete /yes >nul 2>&1
> 
> REM Mapper le lecteur réseau personnel (H:)
> net use H: "\\Serveur\Users$\%USERNAME%" /persistent:no
> 
> REM Mapper le lecteur partagé de l'entreprise (P:)
> net use P: "\\Serveur\Partage$\Entreprise" /persistent:no
> 
> REM Mapper le lecteur du département selon le groupe AD
> REM Vérifier l'appartenance au groupe
> whoami /groups | find "CN=GRP_Comptabilite" >nul
> if %errorlevel% equ 0 (
>     net use K: "\\Serveur\Partage$\Comptabilite" /persistent:no
> )
> 
> whoami /groups | find "CN=GRP_Commercial" >nul
> if %errorlevel% equ 0 (
>     net use K: "\\Serveur\Partage$\Commercial" /persistent:no
> )
> 
> REM Afficher un message de bienvenue
> msg %USERNAME% "Bienvenue %USERNAME% ! Vos lecteurs réseau ont été configurés."
> ```

> [!example] Script PowerShell - Configuration environnement utilisateur
> ```powershell
> # Script d'ouverture de session utilisateur
> # Configure l'environnement de travail
> 
> # Journalisation de l'ouverture de session
> $LogFile = "\\Serveur\Logs$\Connexions\$env:USERNAME.log"
> $LogEntry = "$(Get-Date -Format 'yyyy-MM-dd HH:mm:ss') - Connexion depuis $env:COMPUTERNAME"
> Add-Content -Path $LogFile -Value $LogEntry -ErrorAction SilentlyContinue
> 
> # Mappage des lecteurs réseau
> $Mappings = @{
>     "H:" = "\\Serveur\Users$\$env:USERNAME"
>     "P:" = "\\Serveur\Partage$\Entreprise"
>     "S:" = "\\Serveur\Outils$\Scripts"
> }
> 
> foreach ($Drive in $Mappings.Keys) {
>     # Supprimer si existe déjà
>     if (Test-Path $Drive) {
>         Remove-PSDrive -Name $Drive.TrimEnd(':') -Force -ErrorAction SilentlyContinue
>     }
>     # Mapper le lecteur
>     New-PSDrive -Name $Drive.TrimEnd(':') -PSProvider FileSystem -Root $Mappings[$Drive] -Persist -ErrorAction SilentlyContinue
> }
> 
> # Créer un raccourci sur le bureau vers le partage commun
> $WshShell = New-Object -ComObject WScript.Shell
> $Shortcut = $WshShell.CreateShortcut("$env:USERPROFILE\Desktop\Dossiers partagés.lnk")
> $Shortcut.TargetPath = "\\Serveur\Partage$\Entreprise"
> $Shortcut.IconLocation = "%SystemRoot%\System32\imageres.dll,3"
> $Shortcut.Description = "Accès aux dossiers partagés de l'entreprise"
> $Shortcut.Save()
> 
> # Synchroniser l'heure du client
> w32tm /resync /nowait | Out-Null
> ```

### Scripts de fermeture de session

> [!info] Emplacement
> **Configuration utilisateur** → Stratégies → Paramètres Windows → Scripts (ouverture/fermeture de session) → Fermeture de session

> [!tip] Cas d'usage typiques
> - **Nettoyage du bureau** et des fichiers temporaires
> - **Sauvegarde** de fichiers utilisateur modifiés
> - **Déconnexion de lecteurs** réseau
> - **Journalisation** de la déconnexion
> - **Suppression de fichiers** en cache

> [!example] Script de fermeture de session - Nettoyage et journalisation
> ```batch
> @echo off
> REM Script de fermeture de session utilisateur
> REM Nettoie l'environnement et journalise la déconnexion
> 
> REM Journalisation de la déconnexion
> echo %DATE% %TIME% - Déconnexion de %USERNAME% depuis %COMPUTERNAME% >> \\Serveur\Logs$\Deconnexions\%USERNAME%.log
> 
> REM Nettoyage des fichiers temporaires utilisateur
> del /q /f /s "%TEMP%\*.*" >nul 2>&1
> del /q /f /s "%LOCALAPPDATA%\Temp\*.*" >nul 2>&1
> 
> REM Nettoyage du cache Internet Explorer
> RunDll32.exe InetCpl.cpl,ClearMyTracksByProcess 8
> 
> REM Déconnexion des lecteurs réseau (par précaution)
> net use * /delete /yes >nul 2>&1
> 
> REM Suppression des raccourcis temporaires du bureau
> del "%USERPROFILE%\Desktop\*_temp.lnk" >nul 2>&1
> ```

### Bonnes pratiques pour les scripts

> [!tip] Recommandations essentielles

#### 1. Emplacement des scripts

```
Structure recommandée :
\\Domaine.local\SYSVOL\Domaine.local\Policies\{GPO-GUID}\Machine\Scripts\Startup
\\Domaine.local\SYSVOL\Domaine.local\Policies\{GPO-GUID}\Machine\Scripts\Shutdown
\\Domaine.local\SYSVOL\Domaine.local\Policies\{GPO-GUID}\User\Scripts\Logon
\\Domaine.local\SYSVOL\Domaine.local\Policies\{GPO-GUID}\User\Scripts\Logoff

OU créer un partage centralisé :
\\Serveur\Scripts$\Startup
\\Serveur\Scripts$\Shutdown
\\Serveur\Scripts$\Logon
\\Serveur\Scripts$\Logoff
```

> [!info] Utiliser SYSVOL
> Les scripts stockés dans SYSVOL sont **automatiquement répliqués** sur tous les contrôleurs de domaine, assurant la haute disponibilité.

#### 2. Gestion des erreurs et journalisation

```batch
@echo off
REM Toujours ajouter de la journalisation

REM Définir le fichier de log
set LOGFILE=\\Serveur\Logs$\Scripts\%COMPUTERNAME%_%USERNAME%.log

REM Écrire dans le log
echo ============================================ >> %LOGFILE%
echo Script exécuté le %DATE% à %TIME% >> %LOGFILE%
echo Utilisateur : %USERNAME% >> %LOGFILE%
echo Ordinateur : %COMPUTERNAME% >> %LOGFILE%
echo ============================================ >> %LOGFILE%

REM Votre code ici...

REM Capturer les erreurs
if %errorlevel% neq 0 (
    echo ERREUR : Le script a échoué avec le code %errorlevel% >> %LOGFILE%
) else (
    echo SUCCÈS : Script exécuté avec succès >> %LOGFILE%
)
```

#### 3. Gestion du timeout

> [!warning] Durée d'exécution limitée
> Par défaut, Windows attend :
> - **Scripts démarrage/arrêt** : 10 minutes maximum
> - **Scripts ouverture/fermeture** : 10 minutes maximum (peut être modifié)
> 
> Si vos scripts prennent plus de temps, configurez :
> ```
> Configuration ordinateur/utilisateur 
>   → Modèles d'administration 
>     → Système 
>       → Scripts 
>         → Durée maximale d'attente pour les scripts de stratégie de groupe
> ```

#### 4. Ordre d'exécution des scripts

Si plusieurs scripts sont configurés, ils s'exécutent dans l'ordre défini dans la GPO (boutons "Monter" / "Descendre").

> [!tip] Script "maître"
> Plutôt que d'ajouter 10 scripts dans la GPO, créez **un seul script maître** qui appelle les autres :
> 
> ```batch
> @echo off
> REM Script maître d'ouverture de session
> 
> call \\Serveur\Scripts$\Logon\01_mappage_lecteurs.bat
> call \\Serveur\Scripts$\Logon\02_configuration_imprimantes.bat
> call \\Serveur\Scripts$\Logon\03_synchronisation.bat
> call \\Serveur\Scripts$\Logon\04_message_bienvenue.bat
> ```

#### 5. Sécurité des scripts

> [!warning] Scripts et sécurité
> - **Ne jamais inclure** de mots de passe ou identifiants en clair dans les scripts
> - Utiliser les **autorisations NTFS** pour limiter l'accès aux scripts sensibles
> - Préférer **PowerShell** avec signature de scripts (Execution Policy)
> - Journaliser les exécutions pour l'audit

#### 6. PowerShell vs Batch

| Critère | Batch (.bat, .cmd) | PowerShell (.ps1) |
|---------|-------------------|-------------------|
| **Syntaxe** | Simple, limitée | Riche, orientée objet |
| **Gestion d'erreurs** | Basique | Avancée (try/catch) |
| **Fonctionnalités** | Limitées | Très étendues |
| **Compatibilité** | Excellente (ancien) | Moderne (Windows 7+) |
| **Sécurité** | Aucune signature | Signature et Execution Policy |
| **Performance** | Rapide pour tâches simples | Meilleur pour tâches complexes |

> [!tip] Recommandation
> - **Batch** : scripts simples, compatibilité maximale
> - **PowerShell** : scripts complexes, gestion avancée, environnements modernes

---

## 🎨 Préférences de stratégie de groupe

> [!abstract] Flexibilité de configuration
> Les Préférences de stratégie de groupe offrent une approche plus flexible que les stratégies traditionnelles, permettant des configurations qui peuvent être modifiées par l'utilisateur.

### Différence entre Stratégies et Préférences

| Critère | Stratégies (Policies) | Préférences (Preferences) |
|---------|----------------------|---------------------------|
| **Application** | Imposée et verrouillée | Configurée mais modifiable |
| **Modification utilisateur** | ❌ Impossible | ✅ Possible (selon configuration) |
| **Persistance** | Révoquée si GPO supprimée | ✅ Reste en place |
| **Icône dans GPO** | Icône standard | Icône avec flèche verte |
| **Cas d'usage** | Paramètres de sécurité obligatoires | Configurations par défaut, personnalisation |

> [!example] Exemple concret de la différence
> **Stratégie** - Fond d'écran d'entreprise :
> - Le fond d'écran est imposé
> - L'utilisateur **ne peut pas** le changer
> - Si la GPO est supprimée, le paramètre **revient à la valeur par défaut**
> 
> **Préférence** - Fond d'écran d'entreprise :
> - Le fond d'écran est configuré au premier démarrage
> - L'utilisateur **peut** le changer s'il le souhaite
> - Si la GPO est supprimée, le fond d'écran actuel **reste en place**

> [!warning] Quand utiliser les préférences ?
> ✅ **Utiliser les préférences pour** :
> - Configurations qui peuvent être personnalisées par l'utilisateur
> - Paramètres par défaut (valeurs initiales)
> - Configurations non critiques pour la sécurité
> 
> ❌ **Ne PAS utiliser les préférences pour** :
> - Paramètres de sécurité critiques
> - Configurations qui DOIVENT rester immuables
> - Restrictions d'accès ou de droits

### Types de préférences disponibles

> [!info] Emplacement
> **Configuration ordinateur** → Préférences → Paramètres Windows / Paramètres du Panneau de configuration
> **Configuration utilisateur** → Préférences → Paramètres Windows / Paramètres du Panneau de configuration

#### Préférences Windows (Configuration ordinateur)

| Catégorie | Éléments configurables |
|-----------|------------------------|
| **Applications** | Installation d'applications, configuration |
| **Mappages de lecteurs** | Création, modification de lecteurs réseau |
| **Variables d'environnement** | Création/modification de variables système |
| **Fichiers** | Copie, suppression, remplacement de fichiers |
| **Dossiers** | Création, suppression, modification de dossiers |
| **Fichiers INI** | Modification de fichiers .ini |
| **Raccourcis** | Création de raccourcis (.lnk) |
| **Services** | Configuration de services Windows |
| **Tâches planifiées** | Création de tâches programmées |

#### Préférences Windows (Configuration utilisateur)

Mêmes catégories que Configuration ordinateur, mais appliquées dans le contexte utilisateur.

#### Préférences du Panneau de configuration

| Catégorie | Éléments configurables |
|-----------|------------------------|
| **Options des dossiers** | Affichage, extensions, fichiers cachés |
| **Imprimantes** | Connexion, configuration d'imprimantes réseau |
| **Options d'alimentation** | Schémas d'alimentation |
| **Options régionales** | Langue, format date/heure |
| **Périphériques** | Configuration de périphériques |
| **Tâches planifiées** | Tâches automatisées |

> [!example] Exemple pratique - Créer un raccourci sur tous les bureaux
> ```
> Configuration utilisateur 
>   → Préférences 
>     → Paramètres Windows 
>       → Raccourcis
> 
> Action : Créer
> Nom : Portail Intranet Entreprise
> Emplacement : Bureau
> Chemin cible : https://intranet.entreprise.local
> Icône : %SystemRoot%\System32\imageres.dll,18
> 
> ✅ L'utilisateur pourra supprimer ce raccourci s'il le souhaite
> ✅ Le raccourci sera recréé uniquement si l'action est définie sur "Remplacer" et que le ciblage est actif
> ```

> [!example] Exemple pratique - Mapper un lecteur réseau avec préférences
> ```
> Configuration utilisateur
>   → Préférences
>     → Paramètres Windows
>       → Mappages de lecteurs
> 
> Action : Créer
> Emplacement : \\Serveur\Partage$\Commun
> Lettre de lecteur : P:
> Libellé : Partage Entreprise
> Reconnecter : Activé
> 
> ✅ Plus flexible qu'un script
> ✅ L'utilisateur peut déconnecter le lecteur s'il le souhaite
> ✅ Il sera reconnecté à la prochaine ouverture de session si toujours actif
> ```

### Actions des préférences

Les préférences supportent 4 actions différentes :

| Action | Description | Comportement |
|--------|-------------|--------------|
| **Créer** | Crée l'élément s'il n'existe pas | Ne fait rien si l'élément existe déjà |
| **Remplacer** | Supprime et recrée l'élément | Écrase l'élément existant à chaque application de la GPO |
| **Mettre à jour** | Met à jour uniquement les propriétés définies | Crée si n'existe pas, modifie sinon sans tout écraser |
| **Supprimer** | Supprime l'élément | Supprime l'élément s'il existe |

> [!tip] Quelle action choisir ?
> - **Créer** : Pour une configuration initiale qui ne doit être faite qu'une fois
> - **Remplacer** : Pour forcer une configuration à chaque rafraîchissement (comme une stratégie)
> - **Mettre à jour** : Pour ajuster certaines propriétés sans tout écraser (recommandé)
> - **Supprimer** : Pour nettoyer une ancienne configuration

### Ciblage des préférences

> [!important] Puissance du ciblage
> Le ciblage permet d'appliquer des préférences **uniquement à certains utilisateurs/ordinateurs** selon des critères précis, sans créer de multiples GPO.

> [!info] Activer le ciblage
> Clic droit sur une préférence → **Ciblage...** → Cocher "Ciblage au niveau de l'élément"

#### Critères de ciblage disponibles

| Critère | Description | Exemple d'usage |
|---------|-------------|-----------------|
| **Appartenance à un groupe de sécurité** | L'utilisateur/ordinateur est membre d'un groupe AD | Mapper un lecteur uniquement pour les commerciaux |
| **Unité d'organisation** | L'objet est dans une OU spécifique | Appliquer seulement aux ordinateurs de l'OU "Portables" |
| **Système d'exploitation** | Version de Windows | Configurer uniquement Windows 10/11 |
| **Adresse IP** | Plage d'adresses IP | Différencier siège social et agences |
| **Site Active Directory** | Site AD de l'ordinateur | Configuration par localisation géographique |
| **Variable d'environnement** | Valeur d'une variable système | Différencier selon %PROCESSOR_ARCHITECTURE% |
| **Type de processeur** | Architecture du processeur | x86 vs x64 |
| **Mémoire RAM** | Quantité de RAM | Optimisations selon les ressources |
| **Espace disque** | Espace disponible sur un disque | Actions de nettoyage si < 10 Go |
| **Connexion réseau** | Type de connexion | Câble vs Wi-Fi |
| **Portable ou fixe** | Type d'ordinateur | Paramètres d'alimentation différents |
| **Langue du système** | Langue d'affichage de Windows | Configuration régionale |
| **Heure** | Plage horaire | Scripts uniquement pendant les heures de bureau |

> [!example] Exemple de ciblage complexe - Lecteur réseau pour commerciaux itinérants
> ```
> Préférence : Mappage de lecteur K: vers \\Serveur\Commercial$
> 
> Ciblage :
> ┌─ ET (AND)
> │  ├─ Groupe de sécurité : GRP_Commercial
> │  ├─ Type d'ordinateur : Portable
> │  └─ OU (OR)
> │     ├─ Connexion réseau : Connexion au domaine
> │     └─ Adresse IP : Dans la plage 192.168.1.0/24
> 
> Résultat : 
> Le lecteur K: sera mappé uniquement pour les utilisateurs du groupe
> GRP_Commercial, utilisant un ordinateur portable, ET connectés au 
> réseau du domaine (soit par câble au bureau, soit via VPN).
> ```

> [!tip] Opérateurs logiques dans le ciblage
> - **ET (AND)** : Toutes les conditions doivent être vraies
> - **OU (OR)** : Au moins une condition doit être vraie
> - **NON (NOT)** : Inverse la condition
> 
> Vous pouvez imbriquer des groupes de conditions pour créer des logiques complexes !

> [!warning] Performance du ciblage
> Un ciblage trop complexe peut ralentir l'application de la GPO. Privilégiez :
> - Le ciblage par groupe de sécurité (le plus performant)
> - Des conditions simples et claires
> - Évitez les imbrications excessives (> 3 niveaux)

---

## 📦 Installation de logiciels via GPO

> [!abstract] Déploiement centralisé d'applications
> Les GPO permettent de déployer automatiquement des logiciels sur les ordinateurs et utilisateurs du domaine, garantissant un parc logiciel homogène et à jour.

> [!info] Emplacement
> **Configuration ordinateur** → Stratégies → Paramètres du logiciel → Installation de logiciel
> **Configuration utilisateur** → Stratégies → Paramètres du logiciel → Installation de logiciel

### Méthodes de déploiement

#### 1. Attribution vs Publication

| Critère | Attribution (Assigned) | Publication (Published) |
|---------|------------------------|-------------------------|
| **Disponible pour** | Ordinateurs ET Utilisateurs | **Utilisateurs uniquement** |
| **Installation** | Automatique et obligatoire | Facultative (Ajout/Suppression de programmes) |
| **Moment** | Au démarrage (ordinateur) ou ouverture de session (utilisateur) | Quand l'utilisateur le demande |
| **Affichage** | Non visible (installé en arrière-plan) | Visible dans "Programmes et fonctionnalités" |
| **Désinstallation utilisateur** | ❌ Impossible | ✅ Possible |
| **Icône sur bureau** | Créée automatiquement | Créée si l'utilisateur installe |

> [!example] Quand utiliser chaque méthode ?
> **Attribution** :
> ```
> ✅ Logiciels obligatoires et critiques :
>    - Antivirus
>    - Suite Office
>    - Client VPN
>    - Outils de monitoring
> 
> ✅ Logiciels devant être disponibles immédiatement
> ✅ Applications nécessaires au fonctionnement du poste
> ```
> 
> **Publication** :
> ```
> ✅ Logiciels optionnels :
>    - Adobe Reader
>    - Logiciels métiers spécifiques
>    - Outils de développement
>    - Utilitaires complémentaires
> 
> ✅ Applications que seuls certains utilisateurs utiliseront
> ✅ Logiciels non critiques
> ```

#### 2. Installation par ordinateur vs utilisateur

| | Configuration ordinateur | Configuration utilisateur |
|---|-------------------------|---------------------------|
| **Installation** | Pour tous les utilisateurs de l'ordinateur | Uniquement pour l'utilisateur ciblé |
| **Contexte** | SYSTEM (droits élevés) | Utilisateur (droits limités) |
| **Emplacement** | Program Files | Profil utilisateur |
| **Persistance** | Reste même si utilisateur change | Suit l'utilisateur sur d'autres postes |
| **Méthode** | Attribution uniquement | Attribution OU Publication |

> [!tip] Recommandation générale
> Privilégier **Configuration ordinateur** pour :
> - Applications nécessitant des droits admin
> - Logiciels devant être disponibles pour tous les utilisateurs
> - Meilleure performance (installation une fois, pas par utilisateur)

### Packages MSI et transformation

> [!important] Format MSI obligatoire
> Le déploiement via GPO nécessite des packages au format **Windows Installer (.msi)**. Les fichiers .exe ne sont pas supportés nativement.

#### Créer un partage pour les packages

```powershell
# Sur le serveur de fichiers, créer un partage pour les MSI

# Créer le dossier
New-Item -Path "D:\Applications" -ItemType Directory

# Créer un partage en lecture seule
New-SmbShare -Name "Applications$" -Path "D:\Applications" -ReadAccess "Utilisateurs du domaine" -FullAccess "Admins du domaine"

# Structure recommandée :
# D:\Applications\
#   ├─ Office365\
#   │  └─ Office365ProPlus.msi
#   ├─ AdobeReader\
#   │  └─ AdobeReader_x64.msi
#   ├─ 7-Zip\
#   │  └─ 7z-x64.msi
#   └─ Antivirus\
#      └─ Kaspersky.msi
```

> [!warning] Autorisations critiques
> Les ordinateurs doivent pouvoir accéder au partage :
> - Ajouter **"Ordinateurs du domaine"** avec autorisation en **Lecture**
> - Ou bien le groupe **"Utilisateurs authentifiés"**

#### Fichiers de transformation (.mst)

Les fichiers de transformation (.mst) permettent de **personnaliser l'installation** d'un MSI sans modifier le package original.

> [!example] Exemple - Personnalisation de Microsoft Office
> ```
> Package de base : Office365ProPlus.msi
> Transformation : Office365_Config_Standard.mst
> 
> Personnalisations dans le .mst :
> ✅ Installation silencieuse (aucune interface)
> ✅ Désactiver les mises à jour automatiques (gérées par GPO)
> ✅ Définir le serveur KMS pour l'activation
> ✅ Installer uniquement Word, Excel, PowerPoint, Outlook
> ✅ Ne pas installer OneDrive
> ✅ Paramétrer Outlook avec le serveur Exchange interne
> ✅ Accepter automatiquement la licence
> ```

> [!info] Créer des fichiers .mst
> Utilisez l'outil **Office Customization Tool** (OCT) pour Office, ou **Orca** (SDK Windows) pour les autres MSI.

### Configuration du déploiement

> [!example] Déployer une application - Étapes complètes

#### 1. Préparation

```
1. Obtenir le package MSI (et .mst si nécessaire)
2. Copier dans le partage \\Serveur\Applications$\NomApplication\
3. Tester l'installation manuellement :
   msiexec /i "\\Serveur\Applications$\NomApp\app.msi" /qn
```

#### 2. Configuration dans la GPO

```
Édition de la GPO :
├─ Configuration ordinateur (ou utilisateur)
│  └─ Stratégies
│     └─ Paramètres du logiciel
│        └─ Installation de logiciel
│           └─ Clic droit → Nouveau → Package...

Sélection du package :
✅ Parcourir vers : \\Serveur\Applications$\NomApp\app.msi
⚠️ Utiliser le chemin UNC (\\Serveur\...), PAS de lecteur mappé (Z:\)

Méthode de déploiement :
○ Attribué (Assigned) - Installation automatique obligatoire
○ Publié (Published) - Installation facultative [utilisateur uniquement]
○ Avancé - Configuration détaillée

Recommandation : Choisir "Avancé" pour plus d'options
```

#### 3. Configuration avancée

**Onglet Déploiement** :

| Option | Description | Recommandation |
|--------|-------------|----------------|
| **Type de déploiement** | Attribué ou Publié | Attribué pour logiciels critiques |
| **Options de déploiement** | | |
| └ Installer automatiquement par extension de fichier | Installe si un fichier du type est ouvert | ✅ Activé (pratique) |
| └ Désinstaller cette application si elle ne peut plus être gérée | Supprime si GPO retirée | ⚠️ Selon politique |
| └ Ne pas afficher ce package dans "Ajout/Suppression" | Cache l'application | ❌ Désactivé (transparence) |
| └ Installer l'application à l'ouverture de session | Installation immédiate | ✅ Si critique |
| **Options d'installation** | | |
| └ Interface utilisateur de base | Affiche progression uniquement | ✅ Recommandé |
| └ Maximum | Affiche toutes les fenêtres | ❌ Non (intervention utilisateur) |

**Onglet Mises à niveau** :

Permet de gérer les mises à jour d'applications déjà déployées.

```
Scénario : Mise à jour de Adobe Reader 2022 vers 2024

1. Déployer le nouveau package (AdobeReader2024.msi)
2. Dans les propriétés de AdobeReader2024 :
   Onglet Mises à niveau → Ajouter...
   ├─ Package à mettre à niveau : AdobeReader2022
   ├─ Désinstaller l'application existante, puis installer le package de mise à niveau
   └─ ✅ OK

Résultat :
- Adobe Reader 2022 sera automatiquement désinstallé
- Adobe Reader 2024 sera installé automatiquement
- Les paramètres utilisateur seront préservés (si pris en charge par l'application)
```

**Onglet Catégories** :

Permet de classer les applications dans le Panneau de configuration.

```
Créer des catégories personnalisées :
- Bureautique
- Multimédia
- Développement
- Utilitaires
- Sécurité

Avantage : Facilite la recherche pour les utilisateurs dans "Programmes et fonctionnalités"
```

**Onglet Modifications** :

Pour ajouter des fichiers de transformation (.mst).

```
Ajouter... → Sélectionner le fichier .mst
Ordre d'application : du haut vers le bas

Exemple :
1. Office_Config_Base.mst
2. Office_Config_Service_Compta.mst
```

### Gestion des mises à jour et désinstallations

#### Mettre à jour une application

**Méthode 1 : Mise à niveau automatique** (recommandée)

```
1. Déployer le nouveau package MSI dans une nouvelle GPO ou dans la même
2. Configurer l'onglet "Mises à niveau" pour désinstaller l'ancienne version
3. Appliquer la GPO
4. Au prochain redémarrage/ouverture de session :
   - Ancienne version désinstallée
   - Nouvelle version installée
```

**Méthode 2 : Réaffectation** (pour corrections/reconfigurations)

```
Dans la GPO actuelle :
Clic droit sur le package → Toutes les tâches → Réaffecter l'application

Effet :
- Force la réinstallation ou la réparation
- Réapplique le package MSI
- Utile si l'installation est corrompue
```

#### Désinstaller une application

**Méthode 1 : Désinstallation automatique**

```
Dans la GPO :
Clic droit sur le package → Toutes les tâches → Supprimer

2 options :
○ Désinstaller immédiatement le logiciel des utilisateurs et des ordinateurs
   → Désinstallation forcée au prochain rafraîchissement de GPO
   
○ Permettre aux utilisateurs de continuer à utiliser le logiciel, mais empêcher les nouvelles installations
   → Désinstallation progressive (ne concerne que les nouvelles installations)

Recommandation : Option 1 pour désinstallation complète et rapide
```

**Méthode 2 : Retirer la GPO**

```
Simplement supprimer ou dissocier la GPO qui déploie l'application

Comportement :
- Si "Désinstaller si ne peut plus être gérée" était activé → Désinstallation automatique
- Sinon → L'application reste installée mais ne sera plus déployée sur de nouveaux postes
```

### Dépannage du déploiement

> [!warning] Problèmes courants et solutions

#### L'application ne s'installe pas

**Vérifications** :

```
✅ Le package MSI est accessible :
   - Tester : \\Serveur\Applications$\App\app.msi depuis un poste client
   - Vérifier les autorisations : "Ordinateurs du domaine" doit avoir accès en lecture

✅ La GPO est bien liée et appliquée :
   - gpresult /h rapport.html
   - Vérifier que la GPO apparaît dans "Paramètres du logiciel appliqués"

✅ Le package MSI n'est pas corrompu :
   - msiexec /i "chemin\app.msi" /qn /l*v C:\log_install.txt
   - Analyser le fichier log

✅ Aucun filtrage de sécurité bloquant :
   - Vérifier l'onglet "Délégation" de la GPO
   - S'assurer que "Utilisateurs authentifiés" ou le groupe cible a bien "Lire" + "Appliquer"

✅ L'utilisateur/ordinateur est dans la bonne OU :
   - Vérifier l'emplacement de l'objet dans AD
```

#### L'installation échoue avec une erreur

**Journaux à consulter** :

```
Journaux Windows :
- Observateur d'événements → Journaux Windows → Application
- Rechercher les événements MsiInstaller (source)

Journaux GPO :
- %SystemRoot%\Debug\UserMode\gpsvc.log (activer le mode debug si nécessaire)

Créer un log d'installation manuel :
msiexec /i "\\Serveur\Applications$\App\app.msi" /qn /l*v C:\Temp\install_debug.log
```

**Erreurs courantes** :

| Code erreur | Signification | Solution |
|-------------|---------------|----------|
| **1603** | Erreur fatale générique | Consulter le log détaillé, souvent droits insuffisants ou conflit logiciel |
| **1619** | Package inaccessible | Vérifier le chemin UNC et les autorisations réseau |
| **1625** | Installation interdite par stratégie | Vérifier les restrictions logicielles / AppLocker |
| **1642** | Redémarrage nécessaire | Un redémarrage en attente bloque l'installation |
| **1722** | Service Windows Installer inaccessible | Vérifier que le service est démarré |

> [!tip] Activation des logs détaillés
> Pour des logs exhaustifs, activer dans la GPO :
> ```
> Configuration ordinateur
>   → Modèles d'administration
>     → Composants Windows
>       → Windows Installer
>         → Activer la journalisation : iwearucmpvox
> ```

---

> [!success] Points clés à retenir
> 
> **Configuration ordinateur vs utilisateur** :
> - Ordinateur = au démarrage, privilèges système, prioritaire
> - Utilisateur = à l'ouverture de session, privilèges limités
> 
> **Sécurité via GPO** :
> - Stratégies de mot de passe complexes et historique
> - Verrouillage de compte contre attaques par force brute
> - Audit pour traçabilité (connexions, modifications, accès)
> - Pare-feu avec profils domaine/privé/public
> 
> **Redirection de dossiers** :
> - Documents, Bureau, Favoris recommandés
> - Serveur de fichiers avec partage caché (\\serveur\share$)
> - Autorisations NTFS : CREATOR OWNER + pas de contrôle total racine
> - Avantages : sauvegarde, mobilité, centralisation
> 
> **Scripts** :
> - 4 types : démarrage, arrêt, ouverture, fermeture de session
> - Démarrage/arrêt = SYSTEM, ouverture/fermeture = utilisateur
> - Toujours journaliser et gérer les erreurs
> - Préférer un script maître appelant les autres
> 
> **Préférences** :
> - Différence avec stratégies : modifiable par utilisateur, persiste après suppression GPO
> - Ciblage puissant : groupes, OU, OS, IP, portable/fixe, etc.
> - Actions : Créer, Remplacer, Mettre à jour, Supprimer
> 
> **Déploiement logiciels** :
> - Format MSI obligatoire, partage UNC accessible
> - Attribution (automatique) vs Publication (facultative)
> - Fichiers .mst pour personnalisation
> - Mise à niveau automatique pour les mises à jour
> - Logs et gpresult pour dépannage

---

**Ce cours couvre l'intégralité de la configuration des GPO pour une gestion professionnelle d'Active Directory.** 🎯