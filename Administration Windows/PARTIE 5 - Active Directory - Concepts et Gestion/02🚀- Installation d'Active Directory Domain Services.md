
## 📋 Table des matières

1. [Prérequis pour un DC](#prérequis-pour-un-dc)
2. [Promotion d'un serveur en DC](#promotion-dun-serveur-en-dc)
3. [Création d'une nouvelle forêt](#création-dune-nouvelle-forêt)
4. [Ajout d'un DC à un domaine existant](#ajout-dun-dc-à-un-domaine-existant)
5. [Niveaux fonctionnels de forêt et domaine](#niveaux-fonctionnels-de-forêt-et-domaine)
6. [DNS et Active Directory](#dns-et-active-directory)

---

## Prérequis pour un DC

Avant de déployer un contrôleur de domaine, il est essentiel de vérifier que le serveur répond à tous les prérequis matériels, logiciels et réseau.

### 💻 Prérequis matériels

Les exigences matérielles varient selon la taille de l'environnement :

| Taille de l'environnement | CPU | RAM | Disque |
|---------------------------|-----|-----|--------|
| **Petit** (< 1000 objets) | 2 cœurs | 4 GB | 40 GB |
| **Moyen** (1000-10000 objets) | 4 cœurs | 8 GB | 80 GB |
| **Grand** (> 10000 objets) | 8+ cœurs | 16+ GB | 120+ GB |

> [!tip] Dimensionnement
> Pour un DC de production, prévoyez toujours plus de ressources que le minimum requis. La base de données AD (NTDS.DIT) grandit avec le temps, et les performances dépendent fortement de la RAM disponible.

**Emplacement des données critiques** :
- **Base de données NTDS.DIT** : disque rapide (SSD recommandé)
- **Logs de transaction** : idéalement sur un disque séparé
- **SYSVOL** : peut être sur le même disque que la base

> [!warning] Disques séparés pour les logs
> Pour optimiser les performances et la récupération en cas de panne, placez les logs de transaction sur un disque physiquement séparé de la base de données.

### 🖥️ Prérequis système

**Version de Windows Server requise** :
- Windows Server 2016 ou ultérieur (2019, 2022)
- Édition Standard ou Datacenter
- Installation avec expérience de bureau ou Server Core

**Configuration système requise** :
```powershell
# Vérifier la version de Windows
Get-ComputerInfo | Select-Object WindowsProductName, WindowsVersion, OsHardwareAbstractionLayer

# Vérifier les ressources
Get-ComputerInfo | Select-Object CsProcessors, CsTotalPhysicalMemory, CsSystemType

# Vérifier l'espace disque disponible
Get-Volume | Where-Object {$_.DriveLetter -eq 'C'} | Select-Object DriveLetter, FileSystemLabel, Size, SizeRemaining
```

### 🌐 Prérequis réseau

**Configuration IP statique obligatoire** :

Un contrôleur de domaine doit TOUJOURS avoir une adresse IP statique, jamais de DHCP.

```powershell
# Vérifier la configuration réseau actuelle
Get-NetIPAddress -AddressFamily IPv4

# Configurer une IP statique
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.1.10 -PrefixLength 24 -DefaultGateway 192.168.1.1

# Configurer les serveurs DNS
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 127.0.0.1, 192.168.1.10
```

> [!warning] DNS critique
> Le DNS est absolument ESSENTIEL pour Active Directory. Un DC doit pointer vers lui-même (127.0.0.1 ou sa propre IP) comme serveur DNS primaire après l'installation.

**Tableau de configuration réseau recommandée** :

| Paramètre | Premier DC | DC supplémentaire |
|-----------|------------|-------------------|
| **IP** | Statique | Statique |
| **DNS primaire** | 127.0.0.1 (après install) | IP du premier DC |
| **DNS secondaire** | IP d'un autre DC (optionnel) | 127.0.0.1 |
| **Passerelle** | IP du routeur | IP du routeur |

**Ports réseau requis** :

Active Directory nécessite plusieurs ports ouverts entre les DC :

| Port | Protocol | Service | Utilisation |
|------|----------|---------|-------------|
| **53** | TCP/UDP | DNS | Résolution de noms |
| **88** | TCP/UDP | Kerberos | Authentification |
| **135** | TCP | RPC | Communication RPC |
| **389** | TCP/UDP | LDAP | Requêtes annuaire |
| **636** | TCP | LDAPS | LDAP sécurisé |
| **3268** | TCP | GC | Catalogue global |
| **3269** | TCP | GC SSL | Catalogue global sécurisé |
| **445** | TCP | SMB | Réplication SYSVOL |
| **49152-65535** | TCP | RPC dynamique | Réplication AD |

> [!info] Pare-feu
> Si un pare-feu est activé entre les DC, assurez-vous que tous ces ports sont ouverts. Pour simplifier, créez une règle autorisant tout le trafic entre les contrôleurs de domaine.

### 📋 Prérequis logiciels

**Nom du serveur** :
- Maximum 15 caractères (limitation NetBIOS)
- Pas de caractères spéciaux (uniquement lettres, chiffres, tirets)
- Différent du nom de domaine
- Éviter les noms génériques (DC1, SERVER1)

```powershell
# Vérifier le nom actuel
$env:COMPUTERNAME

# Renommer le serveur (nécessite un redémarrage)
Rename-Computer -NewName "DC-PARIS-01" -Restart
```

**Membership du groupe de travail** :
- Le serveur doit être dans un groupe de travail (WORKGROUP) pour le premier DC
- Pour un DC supplémentaire, il peut être dans le domaine ou dans un groupe de travail

**Mises à jour Windows** :
- Installer toutes les mises à jour critiques et de sécurité
- Redémarrer le serveur avant l'installation d'AD DS

```powershell
# Vérifier les mises à jour en attente
Get-WindowsUpdate

# Installer les mises à jour (module PSWindowsUpdate requis)
Install-WindowsUpdate -AcceptAll -AutoReboot
```

### ✅ Checklist avant installation

Avant de commencer l'installation, vérifiez cette checklist :

- [ ] Serveur avec ressources suffisantes (CPU, RAM, disque)
- [ ] Windows Server version supportée et à jour
- [ ] Nom du serveur conforme (≤ 15 caractères)
- [ ] IP statique configurée
- [ ] Serveur DNS accessible (si DC additionnel)
- [ ] Ports réseau ouverts
- [ ] Nom de domaine planifié (ex: entreprise.local)
- [ ] Mot de passe DSRM défini et sécurisé
- [ ] Sauvegarde du serveur effectuée
- [ ] Documentation préparée

> [!tip] Environnement de test
> Pour votre première installation, utilisez toujours un environnement de test (machines virtuelles). Une fois maîtrisé, déployez en production.

---

## Promotion d'un serveur en DC

La promotion d'un serveur en contrôleur de domaine consiste à installer le rôle AD DS et à configurer le serveur comme DC. Ce processus peut être effectué via l'interface graphique ou PowerShell.

### 📦 Installation du rôle AD DS

**Méthode 1 : Server Manager (GUI)**

1. Ouvrir **Server Manager**
2. Cliquer sur **Manage** → **Add Roles and Features**
3. **Before You Begin** → Next
4. **Installation Type** → Role-based or feature-based installation → Next
5. **Server Selection** → Sélectionner le serveur local → Next
6. **Server Roles** → Cocher **Active Directory Domain Services**
7. Cliquer sur **Add Features** dans la popup
8. Next → Next → Next → **Install**

**Méthode 2 : PowerShell (recommandée)**

```powershell
# Installation du rôle AD DS avec les outils de gestion
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

# Vérifier l'installation
Get-WindowsFeature -Name AD-Domain-Services
```

> [!info] Temps d'installation
> L'installation du rôle AD DS prend généralement 2-5 minutes. Cela n'installe que les binaires nécessaires, sans configurer le serveur comme DC.

**Résultat après installation** :
```powershell
Success Restart Needed Exit Code      Feature Result
------- -------------- ---------      --------------
True    No             Success        {Active Directory Domain Services, Group P...}
```

### 🎯 Comprendre la promotion

La **promotion** transforme un serveur Windows avec le rôle AD DS installé en contrôleur de domaine fonctionnel.

**Ce qui se passe pendant la promotion** :
1. Création de la base de données NTDS.DIT
2. Configuration du service DNS (si nécessaire)
3. Création du dossier SYSVOL
4. Configuration des services AD (NTDS, KDC, NetLogon)
5. Réplication avec les DC existants (si applicable)
6. Redémarrage automatique du serveur

> [!warning] Processus irréversible
> Une fois promu, un serveur devient un contrôleur de domaine. Le rétrograder (demote) effacera toutes les données AD locales. Assurez-vous de vouloir vraiment promouvoir ce serveur.

### 🔐 Mot de passe DSRM

Le **Directory Services Restore Mode (DSRM)** est un mode de démarrage spécial pour réparer ou restaurer Active Directory.

**Caractéristiques du mot de passe DSRM** :
- Défini UNIQUEMENT lors de la promotion
- Indépendant de tous les comptes AD
- Stocké localement sur le DC
- Utilisé pour se connecter en mode DSRM

```powershell
# Le mot de passe DSRM doit respecter les critères de complexité :
# - Au moins 8 caractères
# - Mélange de majuscules, minuscules, chiffres, caractères spéciaux
# - Différent du mot de passe administrateur

# Exemple de mot de passe fort :
# P@ssw0rd-DSRM-2024!
```

> [!warning] Mot de passe critique
> Ne perdez JAMAIS le mot de passe DSRM ! C'est votre seul moyen d'accéder au serveur en mode restauration. Stockez-le dans un gestionnaire de mots de passe sécurisé.

**Modifier le mot de passe DSRM après la promotion** :
```powershell
# Lancer ntdsutil
ntdsutil

# Dans ntdsutil :
set dsrm password
reset password on server null
# (null signifie le serveur local)

# Entrer le nouveau mot de passe
# Taper quit deux fois pour sortir
```

### 📊 Options de promotion

Lors de la promotion, plusieurs options influencent la configuration du DC :

| Option | Description | Impact |
|--------|-------------|--------|
| **DNS Server** | Installer le rôle DNS | Recommandé pour le premier DC |
| **Global Catalog** | Activer le catalogue global | Automatique pour le premier DC |
| **Read-Only DC (RODC)** | DC en lecture seule | Pour sites distants non sécurisés |
| **Database path** | Emplacement NTDS.DIT | Par défaut : C:\Windows\NTDS |
| **Log path** | Emplacement des logs | Par défaut : C:\Windows\NTDS |
| **SYSVOL path** | Emplacement SYSVOL | Par défaut : C:\Windows\SYSVOL |

> [!tip] Chemins personnalisés
> Pour de meilleures performances, placez la base de données sur un disque SSD dédié et les logs sur un autre disque.

---

## Création d'une nouvelle forêt

La création d'une nouvelle forêt est nécessaire lorsque vous déployez Active Directory pour la première fois dans votre organisation.

### 🌳 Qu'est-ce qu'une nouvelle forêt ?

Créer une nouvelle forêt signifie :
- Créer le **premier DC** de l'organisation
- Définir le **domaine racine** de la forêt
- Établir le **schéma AD** initial
- Configurer le **premier site** AD
- Attribuer tous les **rôles FSMO** au premier DC

> [!info] Première brique
> Le premier DC d'une nouvelle forêt est la brique fondatrice de toute votre infrastructure Active Directory. Planifiez soigneusement son déploiement.

### 🎯 Choix du nom de domaine

Le nom de domaine est crucial et **ne peut pas être changé facilement** après la création.

**Formats de noms possibles** :

| Format | Exemple | Avantages | Inconvénients |
|--------|---------|-----------|---------------|
| **.local** | entreprise.local | Domaine privé, pas de conflit internet | Non recommandé par Microsoft (Bonjour/mDNS) |
| **.lan / .internal** | entreprise.lan | Domaine privé, clair | Non officiels (mais fonctionnent) |
| **Domaine public** | entreprise.com | Cohérent avec site web | Nécessite split-brain DNS |
| **Sous-domaine** | ad.entreprise.com | Sépare AD du domaine public | Plus complexe |

> [!warning] Éviter .local
> Bien que très courant, le TLD .local est déconseillé par Microsoft depuis 2012 car il entre en conflit avec le protocole mDNS utilisé par Apple Bonjour et certains équipements réseau.

**Recommandations Microsoft 2024** :
```
✅ RECOMMANDÉ : entreprise.internal, ad.entreprise.com
⚠️ ACCEPTABLE : entreprise.lan, entreprise.corp
❌ DÉCONSEILLÉ : entreprise.local
```

**Nom NetBIOS** :
- Généré automatiquement à partir du nom DNS
- Maximum 15 caractères
- Exemple : `entreprise.com` → NetBIOS `ENTREPRISE`

### 🚀 Promotion via GUI (Server Manager)

**Étapes de création d'une nouvelle forêt** :

1. **Après installation du rôle AD DS**, cliquer sur le drapeau de notification dans Server Manager
2. Cliquer sur **Promote this server to a domain controller**

3. **Deployment Configuration** :
   - Sélectionner **Add a new forest**
   - Entrer le nom de domaine racine : `entreprise.internal`
   - Next

4. **Domain Controller Options** :
   - Niveau fonctionnel de la forêt : Windows Server 2016 ou ultérieur
   - Niveau fonctionnel du domaine : Windows Server 2016 ou ultérieur
   - Cocher **Domain Name System (DNS) server**
   - Cocher **Global Catalog (GC)**
   - Entrer le mot de passe DSRM (2 fois)
   - Next

5. **DNS Options** :
   - Avertissement sur la délégation DNS (ignorez pour une nouvelle forêt)
   - Next

6. **Additional Options** :
   - Le nom NetBIOS est proposé automatiquement
   - Vérifier et modifier si nécessaire
   - Next

7. **Paths** :
   - Database folder : `C:\Windows\NTDS` (ou chemin personnalisé)
   - Log files folder : `C:\Windows\NTDS` (ou disque séparé)
   - SYSVOL folder : `C:\Windows\SYSVOL`
   - Next

8. **Review Options** :
   - Vérifier tous les paramètres
   - Optionnel : **View script** pour voir le code PowerShell équivalent
   - Next

9. **Prerequisites Check** :
   - Attendre la vérification (1-2 minutes)
   - Vérifier qu'il n'y a pas d'erreurs bloquantes
   - Les avertissements sont généralement acceptables
   - **Install**

10. **Installation et redémarrage** :
    - L'installation prend 5-15 minutes
    - Le serveur redémarre automatiquement
    - Après redémarrage, le DC est opérationnel

> [!tip] Script PowerShell
> Pendant l'étape "Review Options", cliquez sur "View script" pour obtenir le code PowerShell équivalent. Sauvegardez-le pour reproduire la configuration ou automatiser d'autres DC.

### 💻 Promotion via PowerShell

La méthode PowerShell est plus rapide et reproductible :

```powershell
# Définir les paramètres
$DomainName = "entreprise.internal"
$DomainNetBIOSName = "ENTREPRISE"
$ForestMode = "WinThreshold"  # Windows Server 2016
$DomainMode = "WinThreshold"  # Windows Server 2016
$DSRMPassword = ConvertTo-SecureString "P@ssw0rd-DSRM-2024!" -AsPlainText -Force

# Promotion en DC (nouvelle forêt)
Install-ADDSForest `
    -DomainName $DomainName `
    -DomainNetBIOSName $DomainNetBIOSName `
    -ForestMode $ForestMode `
    -DomainMode $DomainMode `
    -InstallDns:$true `
    -CreateDnsDelegation:$false `
    -DatabasePath "C:\Windows\NTDS" `
    -LogPath "C:\Windows\NTDS" `
    -SysvolPath "C:\Windows\SYSVOL" `
    -SafeModeAdministratorPassword $DSRMPassword `
    -Force:$true `
    -NoRebootOnCompletion:$false
```

> [!warning] Redémarrage automatique
> Le paramètre `-NoRebootOnCompletion:$false` provoque un redémarrage automatique. Sauvegardez tout travail en cours avant de lancer la commande.

**Script avec demande interactive du mot de passe DSRM** :
```powershell
# Installation de la forêt avec demande de mot de passe
Install-ADDSForest `
    -DomainName "entreprise.internal" `
    -DomainNetBIOSName "ENTREPRISE" `
    -ForestMode "WinThreshold" `
    -DomainMode "WinThreshold" `
    -InstallDns:$true `
    -Force:$true

# Le système demandera le mot de passe DSRM de manière sécurisée
```

### ✅ Vérification après installation

Après le redémarrage, vérifiez que le DC fonctionne correctement :

```powershell
# Vérifier le statut du DC
Get-ADDomainController

# Vérifier les rôles FSMO (le premier DC doit tous les avoir)
Get-ADDomain | Select-Object PDCEmulator, RIDMaster, InfrastructureMaster
Get-ADForest | Select-Object DomainNamingMaster, SchemaMaster

# Vérifier le service DNS
Get-Service -Name DNS

# Vérifier la réplication (devrait être vide pour le premier DC)
repadmin /replsummary

# Tester la connectivité DNS
nslookup entreprise.internal
nslookup _ldap._tcp.entreprise.internal

# Vérifier SYSVOL
Get-ChildItem C:\Windows\SYSVOL\sysvol
```

> [!example] Résultat attendu
> ```powershell
> # Get-ADDomainController devrait montrer :
> Name                 : DC-PARIS-01
> Domain               : entreprise.internal
> Forest               : entreprise.internal
> OperatingSystem      : Windows Server 2022
> IsGlobalCatalog      : True
> IsReadOnly           : False
> Enabled              : True
> ```

### 🔍 Résolution de problèmes courants

**Problème 1 : Erreur DNS pendant la promotion**

```
Error: The DNS server is not responding
```

**Solution** :
```powershell
# Vérifier que le service DNS est démarré
Get-Service -Name DNS
Start-Service -Name DNS

# Vérifier la configuration IP
Get-NetIPAddress -AddressFamily IPv4
Get-DnsClientServerAddress

# Pointer temporairement vers un DNS externe pendant la promotion
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 8.8.8.8
```

**Problème 2 : Le serveur ne redémarre pas**

**Solution** :
- Vérifier les logs : `Eventvwr.msc` → Windows Logs → System
- Redémarrer manuellement : `Restart-Computer -Force`
- Vérifier l'état du service NTDS : `Get-Service NTDS`

**Problème 3 : Impossible de se connecter après promotion**

**Solution** :
- Utiliser le compte : `ENTREPRISE\Administrator` (pas Administrator local)
- Ou en format UPN : `Administrator@entreprise.internal`
- Attendre 2-3 minutes après redémarrage (services en cours de démarrage)

---

## Ajout d'un DC à un domaine existant

L'ajout d'un contrôleur de domaine supplémentaire améliore la redondance, la performance et la disponibilité de votre infrastructure AD.

### 🎯 Pourquoi ajouter un DC supplémentaire ?

**Avantages de plusieurs DC** :

✅ **Haute disponibilité** :
- Si un DC tombe, les autres continuent à fonctionner
- Les utilisateurs peuvent toujours s'authentifier
- Pas d'interruption de service

✅ **Répartition de charge** :
- Les requêtes d'authentification sont distribuées
- Amélioration des performances
- Temps de réponse plus rapides

✅ **Continuité géographique** :
- DC dans chaque site pour authentification locale
- Réduction de la latence réseau
- Optimisation de la bande passante WAN

✅ **Tolérance aux pannes** :
- Récupération rapide en cas de panne matérielle
- Pas de perte de données (réplication)
- Maintenance sans interruption

> [!info] Best practice
> Microsoft recommande **au minimum 2 DC** par domaine en production. Pour les grandes organisations, prévoyez 1 DC par site géographique + 1 DC de secours.

### 📋 Prérequis spécifiques

Avant d'ajouter un DC supplémentaire :

**Configuration réseau** :
```powershell
# Le nouveau serveur doit pointer vers un DC existant comme DNS primaire
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.1.10, 127.0.0.1
# 192.168.1.10 = IP du premier DC
# 127.0.0.1 = IP locale (sera utilisée après promotion)
```

**Joindre le domaine (optionnel mais recommandé)** :
```powershell
# Joindre le serveur au domaine avant la promotion
Add-Computer -DomainName "entreprise.internal" -Credential (Get-Credential) -Restart

# Après redémarrage, se connecter avec un compte du domaine
# Format: ENTREPRISE\Administrator
```

> [!tip] Promouvoir depuis le domaine
> Bien qu'il soit possible de promouvoir un serveur qui est dans un groupe de travail, il est plus simple de le joindre au domaine d'abord. Cela évite certaines erreurs de résolution DNS.

### 🚀 Promotion d'un DC supplémentaire via GUI

**Étapes** :

1. Installer le rôle AD DS (comme pour une nouvelle forêt)

2. Lancer la promotion : **Promote this server to a domain controller**

3. **Deployment Configuration** :
   - Sélectionner **Add a domain controller to an existing domain**
   - Domain : `entreprise.internal`
   - Credentials : Cliquer sur **Change** et entrer les identifiants d'un Administrateur du domaine
   - Next

4. **Domain Controller Options** :
   - Sélectionner le **site** où placer le DC (ou laisser par défaut)
   - Cocher **Domain Name System (DNS) server**
   - Cocher **Global Catalog (GC)**
   - Ne PAS cocher Read-only domain controller (sauf besoin spécifique)
   - Entrer le mot de passe DSRM
   - Next

5. **DNS Options** :
   - L'assistant détecte automatiquement les paramètres DNS
   - Next

6. **Additional Options** :
   - **Replicate from** : Sélectionner le DC source pour la réplication
   - Recommandé : **Any domain controller** (sélection automatique)
   - Next

7. **Paths** :
   - Définir les chemins (identiques au premier DC généralement)
   - Next

8. **Review Options** → **Prerequisites Check** → **Install**

9. Redémarrage automatique

### 💻 Promotion d'un DC supplémentaire via PowerShell

```powershell
# Méthode 1 : Depuis un serveur dans le domaine
$DSRMPassword = ConvertTo-SecureString "P@ssw0rd-DSRM-2024!" -AsPlainText -Force

Install-ADDSDomainController `
    -DomainName "entreprise.internal" `
    -InstallDns:$true `
    -CreateDnsDelegation:$false `
    -DatabasePath "C:\Windows\NTDS" `
    -LogPath "C:\Windows\NTDS" `
    -SysvolPath "C:\Windows\SYSVOL" `
    -SafeModeAdministratorPassword $DSRMPassword `
    -Force:$true `
    -NoRebootOnCompletion:$false
```

```powershell
# Méthode 2 : Depuis un serveur en groupe de travail (nécessite credentials)
$DomainCred = Get-Credential -Message "Entrez les credentials d'un admin du domaine"
$DSRMPassword = ConvertTo-SecureString "P@ssw0rd-DSRM-2024!" -AsPlainText -Force

Install-ADDSDomainController `
    -DomainName "entreprise.internal" `
    -Credential $DomainCred `
    -InstallDns:$true `
    -SafeModeAdministratorPassword $DSRMPassword `
    -Force:$true
```

> [!warning] Temps de réplication initial
> Lors de la première réplication, le nouveau DC va copier toute la base de données AD depuis un DC existant. Pour un domaine de plusieurs GB, cela peut prendre 10-30 minutes ou plus.

### 🔄 Réplication initiale

Pendant la promotion, le nouveau DC :

1. **Copie la base NTDS.DIT** depuis un DC source
2. **Réplique SYSVOL** via DFS-R ou FRS
3. **Synchronise le schéma** et la configuration
4. **Récupère le catalogue global** (si GC activé)
5. **Met à jour les enregistrements DNS**

**Suivi de la réplication** :
```powershell
# Pendant la promotion, surveillez depuis le DC source :
Get-ADReplicationPartnerMetadata -Target "DC-PARIS-02" -Scope Domain

# Après la promotion, vérifiez la réplication :
repadmin /replsummary
repadmin /showrepl

# Forcer une réplication complète :
repadmin /syncall /AdeP
```

> [!tip] Réplication SYSVOL
> SYSVOL contient les stratégies de groupe (GPO). Sa réplication peut prendre plus de temps que la base AD. Vérifiez avec :
> ```powershell
> Get-SmbShare | Where-Object {$_.Name -eq "SYSVOL"}
> ```

### 📊 Placement optimal des DC

**Stratégies de déploiement** :

| Scénario | Configuration recommandée |
|----------|---------------------------|
| **Petit bureau unique** | 2 DC dans le même site |
| **Multi-sites** | 1 DC minimum par site |
| **Site distant petit** | 1 RODC (lecture seule) |
| **Datacenter principal** | 2-4 DC + sauvegarde hors-site |
| **Cloud hybride** | 2 DC on-premise + 2 DC Azure |

**Exemple d'architecture multi-sites** :
```
Site Paris (HQ)
├── DC-PARIS-01 (tous les rôles FSMO)
├── DC-PARIS-02 (GC, DNS)
└── DC-PARIS-03 (GC, DNS, backup)

Site Londres
├── DC-LONDON-01 (GC, DNS)
└── DC-LONDON-02 (GC, DNS)

Site Tokyo (petit)
└── DC-TOKYO-01 (RODC)
```

### ✅ Vérification après ajout

```powershell
# Lister tous les DC du domaine
Get-ADDomainController -Filter *

# Vérifier la réplication entre DC
repadmin /replsummary

# Tester la connexion LDAP
Test-ComputerSecureChannel -Verbose

# Vérifier les enregistrements DNS
nslookup -type=SRV _ldap._tcp.entreprise.internal

# Vérifier SYSVOL partagé
Get-SmbShare | Where-Object {$_.Name -eq "SYSVOL"}

# Vérifier que le catalogue global fonctionne
nltest /dsgetdc:entreprise.internal /gc
```

**Test de basculement** :
```powershell
# Depuis un poste client, tester l'authentification sur chaque DC
nltest /sc_query:entreprise.internal

# Arrêter un DC et vérifier que les utilisateurs peuvent toujours se connecter
Stop-Computer -ComputerName DC-PARIS-01 -Force

# Se connecter depuis un poste client
# → Doit réussir en utilisant DC-PARIS-02
```

---

## Niveaux fonctionnels de forêt et domaine

Les niveaux fonctionnels (Functional Levels) déterminent les fonctionnalités Active Directory disponibles et les versions de Windows Server supportées comme DC.

### 📊 Qu'est-ce qu'un niveau fonctionnel ?

Un **niveau fonctionnel** est un paramètre qui :
- Active des fonctionnalités spécifiques d'AD
- Définit la version minimale de Windows Server pour les DC
- S'applique au niveau **domaine** et au niveau **forêt**
- Est **irréversible** (ne peut être abaissé)

> [!info] Deux niveaux distincts
> Le niveau fonctionnel de **domaine** et celui de **forêt** sont indépendants. Une forêt peut avoir un niveau fonctionnel différent de ses domaines (tant que le niveau de forêt ≥ niveau de domaine).

### 🌲 Niveau fonctionnel de forêt (Forest Functional Level)

Le niveau fonctionnel de forêt affecte **toute la forêt** et tous ses domaines.

**Niveaux disponibles** :

| Niveau | Version requise minimum | Année | Fonctionnalités clés |
|--------|-------------------------|-------|----------------------|
| **Windows Server 2016** | Windows Server 2016 | 2016 | Privileged Access Management (PAM) |
| **Windows Server 2012 R2** | Windows Server 2012 R2 | 2013 | Améliorations DFS-R SYSVOL |
| **Windows Server 2012** | Windows Server 2012 | 2012 | Amélioration Kerberos, Claims |
| **Windows Server 2008 R2** | Windows Server 2008 R2 | 2009 | Corbeille AD, Authentication Policies |
| **Windows Server 2008** | Windows Server 2008 | 2008 | DFS-R pour SYSVOL |

> [!warning] Niveaux obsolètes
> Les niveaux antérieurs à 2008 (2003, 2000) ne sont plus supportés et ne doivent plus être utilisés. Microsoft recommande au minimum Windows Server 2012 R2 pour les nouveaux déploiements.

**Fonctionnalités ajoutées par niveau de forêt** :

**Windows Server 2016** :
- Privileged Access Management (PAM) avec Microsoft Identity Manager
- Améliorations des stratégies d'authentification
- Support pour les identités cloud hybrides

**Windows Server 2012 R2** :
- Réplication SYSVOL améliorée via DFS-R
- Optimisations de performance

**Windows Server 2012** :
- Claims-based authentication
- Compound authentication
- Kerberos Armoring (FAST)

**Windows Server 2008 R2** :
- **Corbeille Active Directory** (AD Recycle Bin) ✨
- Authentication Mechanism Assurance
- Managed Service Accounts

### 🏢 Niveau fonctionnel de domaine (Domain Functional Level)

Le niveau fonctionnel de domaine affecte **uniquement le domaine** où il est configuré.

**Niveaux disponibles** (identiques à la forêt) :

| Niveau | Fonctionnalités clés |
|--------|----------------------|
| **Windows Server 2016** | Améliorations PAM, Credential Guard |
| **Windows Server 2012 R2** | Améliorations Group MSA |
| **Windows Server 2012** | DCClone, Group Managed Service Accounts |
| **Windows Server 2008 R2** | Distributed File System Replication, Fine-Grained Password Policies |
| **Windows Server 2008** | AES encryption pour Kerberos |

**Fonctionnalités ajoutées par niveau de domaine** :

**Windows Server 2016** :
- Support pour Windows Hello for Business
- Credential Guard pour les DC
- Améliorations Rolling Credential Guard

**Windows Server 2012 R2** :
- Group Managed Service Accounts (gMSA) amélioré
- Protected Users security group
- Authentication Policies and Silos

**Windows Server 2012** :
- Group Managed Service Accounts (gMSA)
- Virtual Domain Controller cloning
- Kerberos Key Distribution Center (KDC) support for claims

**Windows Server 2008 R2** :
- **Fine-Grained Password Policies** ✨ (différentes politiques de mots de passe pour différents groupes)
- Last Interactive Logon Information
- Advanced Encryption Standard (AES 128 et 256) pour Kerberos

### 🔍 Vérifier les niveaux fonctionnels actuels

```powershell
# Niveau fonctionnel du domaine
Get-ADDomain | Select-Object Name, DomainMode

# Niveau fonctionnel de la forêt
Get-ADForest | Select-Object Name, ForestMode

# Vue complète
Get-ADDomain | Select-Object Name, DomainMode, @{Name="ForestMode";Expression={(Get-ADForest).ForestMode}}
```

**Résultat exemple** :
```
Name                 DomainMode         ForestMode
----                 ----------         ----------
entreprise.internal  Windows2016Domain  Windows2016Forest
```

### ⬆️ Élever le niveau fonctionnel

L'élévation du niveau fonctionnel est **irréversible** et nécessite que tous les DC soient à la version requise.

> [!warning] Action irréversible
> Une fois un niveau fonctionnel élevé, vous ne pouvez PLUS ajouter de DC avec des versions Windows Server antérieures. Planifiez soigneusement avant d'élever.

**Prérequis avant élévation** :
1. Tous les DC doivent être au niveau de la version cible minimum
2. Tous les DC doivent être opérationnels et répliquant correctement
3. Sauvegarde complète de tous les DC recommandée

**Vérifier les versions des DC** :
```powershell
# Lister tous les DC avec leur version
Get-ADDomainController -Filter * | Select-Object Name, OperatingSystem, OperatingSystemVersion | Sort-Object Name

# Exemple de résultat :
# Name         OperatingSystem           OperatingSystemVersion
# ----         ---------------           ----------------------
# DC-PARIS-01  Windows Server 2022       10.0 (20348)
# DC-PARIS-02  Windows Server 2019       10.0 (17763)
# DC-LONDON-01 Windows Server 2016       10.0 (14393)
```

**Élever le niveau fonctionnel de domaine** :

```powershell
# Méthode PowerShell (recommandée)
Set-ADDomainMode -Identity "entreprise.internal" -DomainMode Windows2016Domain

# Le système demandera confirmation
# Confirmez avec Y (Yes)
```

**Élever le niveau fonctionnel de forêt** :

```powershell
# Le niveau de forêt doit être ≤ au niveau de domaine le plus bas
Set-ADForestMode -Identity "entreprise.internal" -ForestMode Windows2016Forest

# Confirmation requise
```

**Méthode GUI** :
1. Ouvrir **Active Directory Domains and Trusts**
2. Clic droit sur le domaine → **Raise Domain Functional Level**
3. Sélectionner le niveau cible → **Raise**
4. Pour la forêt : Clic droit sur **Active Directory Domains and Trusts** (racine) → **Raise Forest Functional Level**

> [!tip] Ordre d'élévation
> Élevez toujours le niveau du domaine AVANT le niveau de la forêt. Le niveau de forêt ne peut être supérieur au niveau du domaine le plus bas.

### 📋 Stratégie de niveau fonctionnel

**Recommandations Microsoft 2024** :

| Scénario | Niveau recommandé |
|----------|-------------------|
| **Nouveau déploiement** | Windows Server 2016 minimum |
| **Environnement moderne** | Windows Server 2019 ou 2022 |
| **Migration en cours** | Niveau actuel jusqu'à fin de migration |
| **Compatibilité cloud** | Windows Server 2016+ pour Azure AD Connect |

**Feuille de route typique** :
```
Déploiement initial → Windows Server 2016
         ↓
    2-3 ans plus tard
         ↓
Migration vers 2019 → Élévation à Windows2019Domain/Forest
         ↓
    2-3 ans plus tard
         ↓
Migration vers 2022 → Élévation à Windows2022Domain/Forest
```

> [!info] Cycle de vie
> Alignez vos niveaux fonctionnels avec le cycle de vie Microsoft :
> - Windows Server 2016 : Support jusqu'en janvier 2027
> - Windows Server 2019 : Support jusqu'en janvier 2029
> - Windows Server 2022 : Support jusqu'en octobre 2031

### 🚫 Erreurs courantes lors de l'élévation

**Erreur 1 : DC avec version incompatible**
```
Error: The requested functional level cannot be set because one or more domain controllers are running an earlier version
```
**Solution** : Identifier et mettre à niveau ou rétrograder le DC obsolète
```powershell
Get-ADDomainController -Filter * | Where-Object {$_.OperatingSystem -like "*2012*"}
# Mettre à niveau ce DC avant d'élever le niveau
```

**Erreur 2 : Tentative d'abaissement**
```
Error: The functional level cannot be lowered
```
**Solution** : Impossible d'abaissement. Restaurer depuis une sauvegarde antérieure si absolument nécessaire.

---

## DNS et Active Directory

Le DNS (Domain Name System) est absolument **critique** pour le fonctionnement d'Active Directory. Sans DNS fonctionnel, AD ne peut pas opérer.

### 🎯 Pourquoi le DNS est essentiel pour AD ?

Active Directory utilise DNS pour :

1. **Localiser les contrôleurs de domaine**
   - Les clients trouvent les DC via des enregistrements SRV
   - Format : `_ldap._tcp.dc._msdcs.entreprise.internal`

2. **Résoudre les noms de domaine**
   - `entreprise.internal` → Adresse IP du DC
   - `DC-PARIS-01.entreprise.internal` → 192.168.1.10

3. **Localiser les services**
   - Catalogue global : `_gc._tcp.entreprise.internal`
   - Kerberos : `_kerberos._tcp.entreprise.internal`

4. **Gérer les sites**
   - Les clients trouvent le DC le plus proche
   - `_ldap._tcp.Paris._sites.entreprise.internal`

> [!warning] DNS = Fondation d'AD
> **Sans DNS fonctionnel, Active Directory ne peut PAS fonctionner.** Les authentifications échoueront, les stratégies de groupe ne s'appliqueront pas, et la réplication entre DC sera impossible.

### 📋 Enregistrements DNS essentiels pour AD

Lorsqu'un DC est promu, il crée automatiquement plusieurs enregistrements DNS :

#### Enregistrements A (Host)

```dns
DC-PARIS-01.entreprise.internal     A    192.168.1.10
entreprise.internal                  A    192.168.1.10
gc._msdcs.entreprise.internal        A    192.168.1.10
```

#### Enregistrements SRV (Service)

Les enregistrements SRV sont cruciaux pour la localisation des services :

| Enregistrement SRV | Service | Port | Utilisation |
|-------------------|---------|------|-------------|
| `_ldap._tcp` | LDAP | 389 | Localiser tout DC |
| `_ldap._tcp.dc._msdcs` | LDAP DC | 389 | Localiser les DC spécifiques |
| `_kerberos._tcp` | Kerberos | 88 | Authentification Kerberos |
| `_kpasswd._tcp` | Kerberos PWD | 464 | Changement de mot de passe |
| `_gc._tcp` | Global Catalog | 3268 | Catalogue global |
| `_ldap._tcp.{site}._sites` | LDAP par site | 389 | DC du site spécifique |

**Exemple d'enregistrement SRV** :
```dns
_ldap._tcp.entreprise.internal  SRV  0 100 389 DC-PARIS-01.entreprise.internal
│      │    │                    │    │ │   │   │
│      │    │                    │    │ │   │   └─ Nom du serveur cible
│      │    │                    │    │ │   └─ Port
│      │    │                    │    │ └─ Poids (weight)
│      │    │                    │    └─ Priorité
│      │    │                    └─ Type d'enregistrement
│      │    └─ Domaine
│      └─ Protocole (TCP)
└─ Service (LDAP)
```

### 🔧 Options DNS lors de l'installation d'AD

Lors de la promotion d'un DC, plusieurs options DNS sont disponibles :

#### Option 1 : Installer le rôle DNS sur le DC (recommandé)

**Avantages** :
- Intégration complète AD ↔ DNS
- Gestion simplifiée (une console)
- Zone DNS intégrée à AD (stockée dans AD)
- Réplication automatique avec AD
- Sécurité renforcée (mises à jour sécurisées)

**Configuration** :
```powershell
# Lors de la promotion, le paramètre -InstallDns:$true installe automatiquement DNS
Install-ADDSForest -DomainName "entreprise.internal" -InstallDns:$true
```

> [!tip] Best practice
> Pour 95% des déploiements AD, installer le rôle DNS sur les DC est la meilleure option. C'est simple, sécurisé et performant.

#### Option 2 : Utiliser un serveur DNS externe

**Quand utiliser** :
- Infrastructure DNS existante importante
- Équipe DNS dédiée distincte de l'équipe AD
- Politiques d'entreprise strictes de séparation des rôles

**Exigences** :
- Le DNS externe doit supporter les enregistrements SRV
- Doit accepter les mises à jour dynamiques (au moins pour la zone AD)
- Doit être hautement disponible

> [!warning] Complexité accrue
> Utiliser un DNS externe ajoute de la complexité. Vous devez gérer les autorisations, les mises à jour, et assurer la coordination entre les équipes DNS et AD.

### 🌐 Zones DNS intégrées à Active Directory

Lorsque DNS est installé sur un DC, vous pouvez utiliser des **zones intégrées à AD** (Active Directory-Integrated Zones).

**Caractéristiques** :

| Fonctionnalité | Zone intégrée AD | Zone standard |
|----------------|------------------|---------------|
| **Stockage** | Base de données AD (NTDS.DIT) | Fichier texte (.dns) |
| **Réplication** | Réplication AD multi-maître | Transfert de zone maître → esclave |
| **Sécurité** | Mises à jour sécurisées uniquement | Moins sécurisé |
| **Performance** | Excellente (cache AD) | Bonne |
| **Haute dispo** | Automatique (multi-maître) | Maître unique |

**Avantages des zones intégrées AD** :

✅ **Réplication multi-maître** :
- Tous les DC DNS peuvent accepter des modifications
- Pas de point unique de défaillance

✅ **Sécurité renforcée** :
- Seuls les ordinateurs joints au domaine peuvent mettre à jour leurs enregistrements
- Protection contre les mises à jour malveillantes

✅ **Réplication optimisée** :
- Suit la topologie de réplication AD
- Compression automatique entre sites

✅ **Gestion simplifiée** :
- Une sauvegarde AD = sauvegarde DNS
- Restauration intégrée

**Créer une zone intégrée AD** :
```powershell
# Créer une zone primaire intégrée à AD
Add-DnsServerPrimaryZone -Name "entreprise.internal" -ReplicationScope "Domain" -DynamicUpdate "Secure"

# -ReplicationScope options :
#   "Forest"  → Réplication dans toute la forêt
#   "Domain"  → Réplication dans le domaine uniquement
#   "Legacy"  → Tous les DC DNS (Windows 2000)
```

### 🔍 Vérification du DNS après installation

Après avoir promu un DC, vérifiez que le DNS fonctionne correctement :

```powershell
# 1. Vérifier que le service DNS est démarré
Get-Service -Name DNS

# 2. Vérifier les zones DNS
Get-DnsServerZone

# 3. Vérifier les enregistrements SRV critiques
nslookup -type=SRV _ldap._tcp.dc._msdcs.entreprise.internal

# Résultat attendu :
# _ldap._tcp.dc._msdcs.entreprise.internal  SRV service location:
#     priority       = 0
#     weight         = 100
#     port           = 389
#     svr hostname   = DC-PARIS-01.entreprise.internal

# 4. Vérifier les enregistrements A
nslookup DC-PARIS-01.entreprise.internal
nslookup entreprise.internal

# 5. Test complet avec dcdiag
dcdiag /test:dns /v

# 6. Vérifier l'enregistrement des DC
nltest /dsgetdc:entreprise.internal /force
```

### 🔄 Mises à jour dynamiques DNS

Les mises à jour dynamiques permettent aux ordinateurs d'enregistrer automatiquement leurs noms et adresses IP dans le DNS.

**Types de mises à jour** :

| Type | Sécurité | Qui peut mettre à jour | Recommandation |
|------|----------|------------------------|----------------|
| **Aucune** | ❌ Aucune | Personne | N'utilisez jamais |
| **Non sécurisée** | ⚠️ Faible | N'importe qui | Évitez (sauf tests) |
| **Sécurisée uniquement** | ✅ Forte | Ordinateurs joints au domaine | **Recommandé** |
| **Non sécurisée et sécurisée** | ⚠️ Moyenne | Ordinateurs joints + DHCP | Acceptable pour DHCP |

**Configuration recommandée** :
```powershell
# Activer les mises à jour dynamiques sécurisées uniquement
Set-DnsServerPrimaryZone -Name "entreprise.internal" -DynamicUpdate "Secure"

# Vérifier la configuration
Get-DnsServerZone -Name "entreprise.internal" | Select-Object ZoneName, DynamicUpdate, IsAutoCreated
```

> [!warning] Mises à jour non sécurisées
> Les mises à jour non sécurisées permettent à n'importe quel ordinateur (même non joint au domaine) de modifier les enregistrements DNS. Cela peut être exploité pour des attaques MITM. Utilisez TOUJOURS des mises à jour sécurisées.

**Scavenging (nettoyage des enregistrements obsolètes)** :

Les enregistrements DNS peuvent devenir obsolètes (ordinateurs supprimés, IP changées). Le scavenging nettoie automatiquement ces enregistrements.

```powershell
# Activer le scavenging sur le serveur DNS
Set-DnsServerScavenging -ScavengingState $true -ScavengingInterval 7.00:00:00 -RefreshInterval 7.00:00:00 -NoRefreshInterval 7.00:00:00 -ApplyOnAllZones

# Activer le scavenging sur une zone spécifique
Set-DnsServerZoneAging -Name "entreprise.internal" -Aging $true -RefreshInterval 7.00:00:00 -NoRefreshInterval 7.00:00:00

# Lancer un nettoyage manuel
Start-DnsServerScavenging -Force
```

**Paramètres de scavenging** :
- **NoRefreshInterval** : Période pendant laquelle un enregistrement ne peut pas être rafraîchi (7 jours par défaut)
- **RefreshInterval** : Période après laquelle un enregistrement peut être supprimé s'il n'est pas rafraîchi (7 jours par défaut)
- **ScavengingInterval** : Fréquence d'exécution du nettoyage (7 jours par défaut)

> [!tip] Période totale
> Un enregistrement sera supprimé après : NoRefreshInterval + RefreshInterval = 14 jours par défaut (si non rafraîchi)

### 🔧 Zones de recherche inversée

Les zones de recherche inversée (Reverse Lookup Zones) permettent de résoudre une adresse IP en nom d'hôte.

**Créer une zone de recherche inversée** :
```powershell
# Pour le réseau 192.168.1.0/24
Add-DnsServerPrimaryZone -NetworkId "192.168.1.0/24" -ReplicationScope "Domain" -DynamicUpdate "Secure"

# Vérifier la zone créée
Get-DnsServerZone | Where-Object {$_.IsReverseLookupZone -eq $true}
```

**Résultat** : Zone nommée `1.168.192.in-addr.arpa`

**Tester la résolution inverse** :
```powershell
nslookup 192.168.1.10

# Résultat attendu :
# Nom :    DC-PARIS-01.entreprise.internal
# Address: 192.168.1.10
```

> [!info] Optionnel mais recommandé
> Les zones de recherche inversée ne sont pas strictement nécessaires pour AD, mais elles sont utiles pour le troubleshooting, les logs, et certaines applications qui vérifient les reverse DNS.

### 🛡️ Sécurisation du DNS

**Best practices de sécurité DNS** :

1. **Désactiver la récursion pour les clients externes** :
```powershell
Set-DnsServerRecursion -Enable $false -SecureResponse $true
```

2. **Limiter les transferts de zone** :
```powershell
Set-DnsServerPrimaryZone -Name "entreprise.internal" -SecureSecondaries "TransferToSecureServers" -SecondaryServers 192.168.1.11
```

3. **Activer DNSSEC** (DNS Security Extensions) :
```powershell
# Signer la zone avec DNSSEC
Add-DnsServerSigningKey -ZoneName "entreprise.internal" -CryptoAlgorithm "RsaSha256" -Type "KeySigningKey"
Add-DnsServerSigningKey -ZoneName "entreprise.internal" -CryptoAlgorithm "RsaSha256" -Type "ZoneSigningKey"
```

4. **Restreindre les mises à jour dynamiques** :
```powershell
Set-DnsServerPrimaryZone -Name "entreprise.internal" -DynamicUpdate "Secure"
```

5. **Activer les logs DNS pour l'audit** :
```powershell
Set-DnsServerDiagnostics -All $true
```

### 🚨 Résolution de problèmes DNS courants

**Problème 1 : Les enregistrements SRV ne sont pas créés**

**Symptômes** :
- Les clients ne trouvent pas les DC
- Erreurs d'authentification

**Diagnostic** :
```powershell
nslookup -type=SRV _ldap._tcp.dc._msdcs.entreprise.internal
# Si aucun résultat → problème
```

**Solutions** :
```powershell
# 1. Forcer l'enregistrement du DC dans le DNS
ipconfig /registerdns

# 2. Redémarrer le service Netlogon
Restart-Service Netlogon

# 3. Vérifier les permissions sur la zone DNS
# Le groupe "Domain Controllers" doit avoir les droits de mise à jour

# 4. Recréer manuellement les enregistrements
nltest /dsregdns
```

**Problème 2 : "DNS server not responding"**

**Solutions** :
```powershell
# 1. Vérifier que le service DNS est démarré
Get-Service -Name DNS
Start-Service -Name DNS

# 2. Vérifier la configuration IP
Get-DnsClientServerAddress

# 3. Tester la connectivité
Test-NetConnection -ComputerName 127.0.0.1 -Port 53

# 4. Vérifier le pare-feu
Get-NetFirewallRule -DisplayName "*DNS*" | Where-Object {$_.Enabled -eq $true}
```

**Problème 3 : La réplication DNS ne fonctionne pas entre DC**

**Solutions** :
```powershell
# 1. Vérifier la réplication AD (DNS en dépend)
repadmin /replsummary

# 2. Forcer la réplication
repadmin /syncall /AdeP

# 3. Vérifier la zone intégrée AD
Get-DnsServerZone | Where-Object {$_.IsDsIntegrated -eq $true}

# 4. Vérifier les enregistrements NS (Name Server)
Get-DnsServerResourceRecord -ZoneName "entreprise.internal" -RRType NS
```

---

## 🎯 Récapitulatif

L'installation d'Active Directory Domain Services nécessite une planification minutieuse et le respect de prérequis stricts :

### ✅ Points clés à retenir

**Prérequis** :
- IP statique OBLIGATOIRE
- Ressources suffisantes (RAM, CPU, disque)
- DNS configuré correctement
- Nom de serveur conforme (≤ 15 caractères)

**Nouvelle forêt** :
- Premier DC de l'organisation
- Choix du nom de domaine (éviter .local)
- Mot de passe DSRM critique et sécurisé
- DNS intégré à AD recommandé

**DC supplémentaire** :
- Améliore la redondance et la performance
- Minimum 2 DC en production
- Pointer vers un DC existant comme DNS
- Vérifier la réplication après promotion

**Niveaux fonctionnels** :
- Déterminent les fonctionnalités disponibles
- Irréversibles (ne peuvent être abaissés)
- Windows Server 2016 minimum recommandé
- Élever domaine AVANT la forêt

**DNS et AD** :
- DNS est ESSENTIEL pour AD
- Zones intégrées AD = best practice
- Mises à jour sécurisées uniquement
- Enregistrements SRV critiques pour la localisation des services

> [!tip] Prochaine étape
> Maintenant que votre infrastructure AD est en place avec des DC fonctionnels, l'étape suivante consiste à créer et gérer les objets Active Directory (utilisateurs, groupes, ordinateurs, OU).

---

*Cours créé pour Obsidian - Active Directory - Partie 5*