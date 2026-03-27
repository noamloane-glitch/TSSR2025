# 

> [!info] Vue d'ensemble
> Les comptes ordinateurs représentent les machines jointes au domaine Active Directory. Cette section couvre la jonction au domaine, la gestion et la maintenance de ces comptes essentiels à l'infrastructure AD.

---

## 📑 Table des matières

- [[#🔗 Jonction au Domaine (Windows 10/11)|Jonction au Domaine (Windows 10/11)]]
- [[#🖥️ Jonction au Domaine (Windows Server)|Jonction au Domaine (Windows Server)]]
- [[#⚙️ Gestion des Comptes Ordinateurs dans AD|Gestion des Comptes Ordinateurs dans AD]]
- [[#🔄 Réinitialisation de Comptes Ordinateurs|Réinitialisation de Comptes Ordinateurs]]
- [[#📦 Déplacement entre OU|Déplacement entre OU]]

---

## 🔗 Jonction au Domaine (Windows 10/11)

La jonction d'un poste de travail Windows au domaine Active Directory est une opération fondamentale qui permet la gestion centralisée de la machine et l'authentification des utilisateurs.

### Prérequis

> [!warning] Vérifications Avant Jonction
> **Réseau :**
> - Connectivité réseau avec le contrôleur de domaine
> - Résolution DNS correcte (pointer vers le DNS du domaine)
> - Ports réseau ouverts (TCP/UDP 389, 636, 88, 53, 445, 135)
> 
> **Système :**
> - Droits administrateur local sur la machine
> - Nom d'ordinateur conforme (15 caractères max, pas de caractères spéciaux)
> - Edition compatible (Professionnel, Entreprise, Education - pas Home)
> 
> **Active Directory :**
> - Compte avec droits de jonction au domaine
> - OU de destination identifiée (optionnel)

```powershell
# Vérifier la résolution DNS du domaine
nslookup entreprise.local

# Vérifier la connectivité avec un DC
Test-Connection -ComputerName dc01.entreprise.local -Count 4

# Vérifier le nom actuel de l'ordinateur
hostname

# Vérifier l'édition Windows
Get-ComputerInfo | Select-Object WindowsProductName, WindowsEditionId
```

> [!tip] Préparation DNS
> **Configuration DNS critique :**
> 
> 1. DNS primaire : Adresse IP du contrôleur de domaine
> 2. DNS secondaire : Deuxième DC ou DNS externe (8.8.8.8)
> 
> Sans DNS correctement configuré, la jonction échouera systématiquement.

### Méthode 1 : Interface Graphique

#### Étapes Détaillées

**1. Accéder aux propriétés système**

```
Méthode rapide :
- Touche Windows + Pause/Arrêt
- OU Clic droit sur "Ce PC" → Propriétés
- OU Paramètres → Système → Informations système
```

**2. Cliquer sur "Modifier les paramètres"**

Section "Nom de l'ordinateur, domaine et paramètres de groupe de travail"

**3. Cliquer sur "Modifier..."**

Dans l'onglet "Nom de l'ordinateur"

**4. Sélectionner "Domaine"**

- Entrer le nom du domaine : `entreprise.local`
- Cliquer sur "OK"

**5. Authentification**

- **Nom d'utilisateur** : Compte avec droits de jonction
  - Format 1 : `administrateur@entreprise.local` (UPN)
  - Format 2 : `ENTREPRISE\administrateur` (NetBIOS)
- **Mot de passe** : Mot de passe du compte

**6. Message de bienvenue**

```
"Bienvenue dans le domaine entreprise.local"
```

**7. Redémarrage obligatoire**

Le système doit redémarrer pour finaliser la jonction.

> [!example] Capture d'écran des Écrans Clés
> **Écran 1 - Propriétés Système :**
> ```
> Nom de l'ordinateur : PC-COMPTA-01
> Groupe de travail : WORKGROUP → [Modifier]
> ```
> 
> **Écran 2 - Modification :**
> ```
> ⚪ Groupe de travail
> ⚫ Domaine : [entreprise.local]
> ```
> 
> **Écran 3 - Authentification :**
> ```
> Nom d'utilisateur : administrateur@entreprise.local
> Mot de passe : ****************
> ```

### Méthode 2 : PowerShell (Recommandée)

```powershell
# Méthode basique - Jonction au domaine
Add-Computer -DomainName "entreprise.local" `
             -Credential (Get-Credential) `
             -Restart

# Méthode avancée - Avec OU spécifique
$domain = "entreprise.local"
$ou = "OU=Workstations,OU=Paris,DC=entreprise,DC=local"
$credential = Get-Credential -Message "Compte avec droits de jonction"

Add-Computer -DomainName $domain `
             -OUPath $ou `
             -Credential $credential `
             -Restart -Force

# Jonction avec nouveau nom d'ordinateur
Add-Computer -DomainName "entreprise.local" `
             -NewName "PC-RH-05" `
             -Credential (Get-Credential) `
             -Restart

# Jonction sans redémarrage immédiat
Add-Computer -DomainName "entreprise.local" `
             -Credential (Get-Credential)
Write-Host "Redémarrez manuellement quand vous êtes prêt" -ForegroundColor Yellow

# Jonction avec serveur DC spécifique
Add-Computer -DomainName "entreprise.local" `
             -Server "dc01.entreprise.local" `
             -Credential (Get-Credential) `
             -Restart
```

> [!tip] Avantages de PowerShell
> - **Automatisation** : Scriptable pour déploiements massifs
> - **OU spécifique** : Place directement le compte dans la bonne OU
> - **Traçabilité** : Peut être journalisé automatiquement
> - **Renommage** : Peut changer le nom simultanément
> - **Flexibilité** : Contrôle total sur le processus

### Méthode 3 : Jonction Offline (DJOIN)

La jonction offline permet de préparer la jonction sans connectivité directe au domaine (utile pour sites distants, déploiements OSD).

```powershell
# ÉTAPE 1 : Sur le contrôleur de domaine ou poste avec RSAT
# Créer le blob de provisioning
djoin.exe /provision `
          /domain entreprise.local `
          /machine PC-DISTANT-01 `
          /savefile C:\Provision\PC-DISTANT-01.txt `
          /machineou "OU=Remote,OU=Workstations,DC=entreprise,DC=local"

# ÉTAPE 2 : Transférer le fichier PC-DISTANT-01.txt vers la machine cible
# Via USB, partage réseau, ou autre moyen

# ÉTAPE 3 : Sur la machine cible (droits admin local)
# Appliquer le provisioning
djoin.exe /requestodj `
          /loadfile C:\Temp\PC-DISTANT-01.txt `
          /windowspath %SystemRoot% `
          /localos

# ÉTAPE 4 : Redémarrer
Restart-Computer
```

> [!info] Quand Utiliser DJOIN ?
> **Cas d'usage :**
> - Déploiement de machines sur sites distants sans DC local
> - Provisioning de masse avec MDT/SCCM
> - Machines dans DMZ avec restrictions réseau
> - Préparation de machines avant livraison
> 
> **Avantages :**
> - Pas besoin de connectivité directe au DC pendant la jonction
> - Création préalable du compte ordinateur avec paramètres précis
> - Sécurisé (blob chiffré)

### Vérification Post-Jonction

```powershell
# Vérifier l'appartenance au domaine
Get-ComputerInfo | Select-Object CsDomain, CsDomainRole

# Vérifier la communication avec le domaine
Test-ComputerSecureChannel -Verbose

# Afficher les informations complètes
systeminfo | findstr /B /C:"Domaine"

# Vérifier via interface graphique
Get-WmiObject -Class Win32_ComputerSystem | Select-Object Domain, PartOfDomain

# Lister les contrôleurs de domaine disponibles
nltest /dclist:entreprise.local

# Vérifier le canal sécurisé avec le DC
nltest /sc_query:entreprise.local
```

> [!success] Signes de Jonction Réussie
> ✅ Message "Bienvenue dans le domaine..."
> ✅ Redémarrage sans erreur
> ✅ Écran de connexion affiche "Autre utilisateur"
> ✅ Possibilité de se connecter avec compte domaine
> ✅ `Test-ComputerSecureChannel` retourne `True`
> ✅ Compte ordinateur visible dans ADUC

### Résolution des Problèmes Courants

> [!warning] Erreurs Fréquentes

**❌ "Le domaine spécifié n'existe pas ou n'est pas accessible"**

```powershell
# Vérifier la résolution DNS
nslookup entreprise.local
nslookup _ldap._tcp.dc._msdcs.entreprise.local

# Tester la connectivité LDAP
Test-NetConnection -ComputerName dc01.entreprise.local -Port 389
Test-NetConnection -ComputerName dc01.entreprise.local -Port 636

# Vérifier la configuration DNS
Get-DnsClientServerAddress -AddressFamily IPv4

# Corriger le DNS si nécessaire
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses "10.0.0.10","10.0.0.11"
```

**❌ "Droits d'accès insuffisants"**

- Vérifier que le compte a le droit de joindre des ordinateurs au domaine
- Par défaut : membres de "Domain Admins" ou "Account Operators"
- Ou délégation spécifique sur l'OU cible

```powershell
# Vérifier les droits du compte (sur DC ou avec RSAT)
dsacls "OU=Workstations,DC=entreprise,DC=local" | findstr "Create Computer"
```

**❌ "Le nom d'ordinateur existe déjà"**

```powershell
# Sur le DC ou poste RSAT - Vérifier l'existence
Get-ADComputer -Identity "PC-COMPTA-01"

# Supprimer l'ancien compte si obsolète
Remove-ADComputer -Identity "PC-COMPTA-01" -Confirm:$false

# Ou réinitialiser le mot de passe du compte
Reset-ComputerMachinePassword -Credential (Get-Credential)
```

**❌ "L'horloge de l'ordinateur client et celle du serveur ne sont pas synchronisées"**

```powershell
# Vérifier l'heure actuelle
Get-Date

# Synchroniser avec le DC
w32tm /resync /force

# Configurer la synchronisation avec le domaine
w32tm /config /syncfromflags:domhier /update
net stop w32time && net start w32time
```

> [!tip] Différence de Temps Maximale
> Kerberos tolère maximum **5 minutes** de décalage entre le poste et le DC.
> Au-delà, l'authentification échoue systématiquement.

**❌ "L'ordinateur n'a pas pu être joint car le quota a été dépassé"**

Par défaut, un utilisateur peut joindre 10 ordinateurs au domaine.

```powershell
# Vérifier le quota actuel (sur DC)
Get-ADObject -Identity "DC=entreprise,DC=local" -Properties ms-DS-MachineAccountQuota | 
    Select-Object -ExpandProperty ms-DS-MachineAccountQuota

# Modifier le quota (sur DC avec droits Domain Admin)
Set-ADObject -Identity "DC=entreprise,DC=local" -Replace @{"ms-DS-MachineAccountQuota"="20"}

# Ou déléguer des droits spécifiques sur une OU (recommandé)
```

### Jonction de Masse (Automatisation)

```powershell
# Script de jonction de masse via CSV
# Fichier CSV : computers.csv
# ComputerName,OUPath
# PC-COMPTA-01,OU=Compta,OU=Workstations,DC=entreprise,DC=local
# PC-RH-01,OU=RH,OU=Workstations,DC=entreprise,DC=local

$computers = Import-Csv -Path "C:\Scripts\computers.csv"
$domain = "entreprise.local"
$credential = Get-Credential -Message "Compte administrateur domaine"

foreach ($computer in $computers) {
    Write-Host "Traitement de $($computer.ComputerName)..." -ForegroundColor Cyan
    
    # Se connecter à distance (nécessite PSRemoting activé)
    Invoke-Command -ComputerName $computer.ComputerName -Credential $localAdminCred -ScriptBlock {
        param($domainName, $ouPath, $domainCred)
        
        Add-Computer -DomainName $domainName `
                     -OUPath $ouPath `
                     -Credential $domainCred `
                     -Force
                     
    } -ArgumentList $domain, $computer.OUPath, $credential
    
    Write-Host "$($computer.ComputerName) - Jonction terminée, redémarrage requis" -ForegroundColor Green
}
```

> [!warning] Prérequis pour Jonction à Distance
> - PowerShell Remoting activé sur machines cibles (`Enable-PSRemoting`)
> - Credentials administrateur local
> - Pare-feu configuré (WinRM ports 5985/5986)
> - Réseau permettant la communication

---

## 🖥️ Jonction au Domaine (Windows Server)

La jonction d'un serveur Windows au domaine suit les mêmes principes que pour les postes clients, mais nécessite des considérations supplémentaires.

### Particularités des Serveurs

> [!info] Différences avec Postes Clients
> **Considérations spécifiques :**
> - Serveurs souvent placés dans OU dédiée (différente des workstations)
> - Souvent configurés avec IP statique (vérifier DNS)
> - Peuvent avoir des rôles spécifiques nécessitant configuration post-jonction
> - Impact plus important en cas de problème
> - Généralement moins de jonctions/déjonctions

### Méthode 1 : Server Manager (Interface Graphique)

#### Étapes avec Server Manager

**1. Ouvrir Server Manager**

Démarrer automatiquement ou via `servermanager.exe`

**2. Cliquer sur le nom du serveur (en haut à droite)**

Ou **Local Server** dans le panneau de gauche

**3. Cliquer sur le nom de workgroup actuel**

Ligne "Computer name" / "Workgroup"

**4. Dans System Properties → "Change"**

**5. Sélectionner "Domain"**

- Entrer : `entreprise.local`
- Authentification avec compte admin domaine

**6. Redémarrer le serveur**

> [!warning] Planification du Redémarrage
> Pour les serveurs de production :
> - Planifier une fenêtre de maintenance
> - Prévenir les utilisateurs
> - Vérifier les services critiques
> - Prévoir un plan de retour arrière

### Méthode 2 : PowerShell (Recommandée pour Serveurs)

```powershell
# Jonction serveur avec OU spécifique
$domain = "entreprise.local"
$ouPath = "OU=Servers,OU=Production,DC=entreprise,DC=local"
$credential = Get-Credential -Message "Administrateur domaine"

Add-Computer -DomainName $domain `
             -OUPath $ouPath `
             -Credential $credential `
             -Restart -Force

# Jonction avec options de serveur
Add-Computer -DomainName "entreprise.local" `
             -OUPath "OU=Servers-SQL,OU=Servers,DC=entreprise,DC=local" `
             -Server "dc01.entreprise.local" `
             -Credential (Get-Credential) `
             -Options JoinWithNewName,AccountCreate `
             -Restart

# Jonction sans redémarrage (pour préparation)
Add-Computer -DomainName "entreprise.local" `
             -OUPath "OU=Servers,DC=entreprise,DC=local" `
             -Credential (Get-Credential)

Write-Host "Serveur prêt à redémarrer. Planifiez le redémarrage." -ForegroundColor Yellow
```

### Méthode 3 : Configuration Initiale Sconfig

Pour Windows Server Core (sans interface graphique), l'outil **Sconfig** simplifie la jonction.

```powershell
# Lancer Sconfig
sconfig

# Navigation dans Sconfig :
# 1. Domaine/Groupe de travail → Option 1
# 2. Sélectionner "D" pour Domaine
# 3. Entrer le nom du domaine : entreprise.local
# 4. Fournir les credentials
# 5. Redémarrer

# Alternative complète en PowerShell pour Server Core
Add-Computer -DomainName "entreprise.local" `
             -Credential (Get-Credential) `
             -Restart -Force
```

### Jonction de Serveur Membre dans Zone Démilitarisée (DMZ)

> [!warning] Serveurs en DMZ
> Les serveurs dans une DMZ nécessitent une configuration réseau spécifique :
> - Pare-feu entre DMZ et réseau interne
> - Restrictions de ports strictes
> - Parfois pas de route directe vers DC

**Configuration DNS critique pour DMZ :**

```powershell
# Configurer le DNS vers le DC interne
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" `
                           -ServerAddresses "10.0.0.10","10.0.0.11"

# Vérifier la résolution
nslookup entreprise.local
nslookup dc01.entreprise.local

# Tester la connectivité sur ports AD requis
Test-NetConnection -ComputerName dc01.entreprise.local -Port 389  # LDAP
Test-NetConnection -ComputerName dc01.entreprise.local -Port 636  # LDAPS
Test-NetConnection -ComputerName dc01.entreprise.local -Port 88   # Kerberos
Test-NetConnection -ComputerName dc01.entreprise.local -Port 53   # DNS
Test-NetConnection -ComputerName dc01.entreprise.local -Port 445  # SMB
```

**Ports requis à ouvrir dans le pare-feu DMZ :**

| Port | Protocole | Service | Direction |
|------|-----------|---------|-----------|
| 53 | TCP/UDP | DNS | Bidirectionnel |
| 88 | TCP/UDP | Kerberos | Bidirectionnel |
| 135 | TCP | RPC Endpoint Mapper | Bidirectionnel |
| 389 | TCP/UDP | LDAP | Bidirectionnel |
| 636 | TCP | LDAPS | Bidirectionnel |
| 445 | TCP | SMB | Bidirectionnel |
| 464 | TCP/UDP | Kerberos Password Change | Bidirectionnel |
| 3268 | TCP | Global Catalog | Bidirectionnel |
| 49152-65535 | TCP | RPC Dynamic Ports | Bidirectionnel |

### Vérifications Post-Jonction Serveur

```powershell
# Vérifier l'appartenance au domaine
Get-ComputerInfo | Select-Object CsDomain, CsDomainRole, CsName

# Tester le canal sécurisé
Test-ComputerSecureChannel -Verbose

# Vérifier les contrôleurs de domaine détectés
nltest /dsgetdc:entreprise.local

# Vérifier la réplication Kerberos
klist purge
klist

# Vérifier les stratégies de groupe appliquées
gpresult /r

# Redémarrage d'un service critique pour validation
Restart-Service -Name "Nom-Du-Service" -Verbose
```

### Configuration Post-Jonction pour Serveurs

```powershell
# Mettre à jour les stratégies de groupe immédiatement
gpupdate /force

# Configurer Windows Update pour utiliser WSUS du domaine (si applicable)
# Les GPO le feront généralement automatiquement

# Synchroniser l'heure avec le domaine
w32tm /config /syncfromflags:domhier /update
w32tm /resync /force

# Vérifier les services critiques
Get-Service | Where-Object {$_.StartType -eq 'Automatic' -and $_.Status -ne 'Running'}

# Activer Remote Desktop si nécessaire (commun pour serveurs)
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' `
                 -Name "fDenyTSConnections" -Value 0

Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
```

### Script de Jonction Serveur Complet

```powershell
<#
.SYNOPSIS
    Script complet de jonction serveur au domaine
.DESCRIPTION
    Effectue toutes les vérifications, la jonction, et la configuration post-jonction
#>

# Configuration
$DomainName = "entreprise.local"
$OUPath = "OU=Servers,OU=Production,DC=entreprise,DC=local"
$DNSServers = @("10.0.0.10", "10.0.0.11")

# Fonction de log
function Write-Log {
    param([string]$Message, [string]$Level = "INFO")
    $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    $logMessage = "$timestamp [$Level] $Message"
    Write-Host $logMessage
    Add-Content -Path "C:\Logs\DomainJoin.log" -Value $logMessage
}

# Créer le dossier de log
New-Item -Path "C:\Logs" -ItemType Directory -Force | Out-Null

Write-Log "=== Début du processus de jonction au domaine ==="

# 1. Vérifier les prérequis
Write-Log "Vérification des prérequis..."

# Vérifier l'édition Windows
$edition = (Get-ComputerInfo).WindowsProductName
Write-Log "Edition Windows : $edition"

# 2. Configurer le DNS
Write-Log "Configuration DNS..."
try {
    $adapter = Get-NetAdapter | Where-Object {$_.Status -eq "Up"} | Select-Object -First 1
    Set-DnsClientServerAddress -InterfaceAlias $adapter.Name -ServerAddresses $DNSServers
    Write-Log "DNS configuré : $($DNSServers -join ', ')" "SUCCESS"
} catch {
    Write-Log "Erreur configuration DNS : $_" "ERROR"
    exit 1
}

# 3. Tester la résolution DNS
Write-Log "Test de résolution DNS..."
try {
    $dnsTest = Resolve-DnsName -Name $DomainName -ErrorAction Stop
    Write-Log "Résolution DNS OK : $DomainName" "SUCCESS"
} catch {
    Write-Log "Échec résolution DNS : $_" "ERROR"
    exit 1
}

# 4. Tester la connectivité DC
Write-Log "Test de connectivité avec le domaine..."
$dcTest = Test-NetConnection -ComputerName $DomainName -Port 389 -WarningAction SilentlyContinue
if ($dcTest.TcpTestSucceeded) {
    Write-Log "Connectivité LDAP OK" "SUCCESS"
} else {
    Write-Log "Échec connectivité LDAP" "ERROR"
    exit 1
}

# 5. Demander les credentials
Write-Log "Demande de credentials administrateur domaine..."
$credential = Get-Credential -Message "Compte administrateur domaine pour jonction"

# 6. Joindre au domaine
Write-Log "Jonction au domaine $DomainName..."
try {
    Add-Computer -DomainName $DomainName `
                 -OUPath $OUPath `
                 -Credential $credential `
                 -Force `
                 -ErrorAction Stop
    
    Write-Log "Jonction réussie !" "SUCCESS"
    
    # 7. Configuration post-jonction
    Write-Log "Configuration post-jonction..."
    
    # Synchronisation horaire
    w32tm /config /syncfromflags:domhier /update
    w32tm /resync /force
    Write-Log "Synchronisation horaire effectuée"
    
    Write-Log "=== Jonction terminée avec succès ==="
    Write-Log "Redémarrage dans 60 secondes..."
    
    shutdown /r /t 60 /c "Redémarrage suite à jonction domaine"
    
} catch {
    Write-Log "Échec de la jonction : $_" "ERROR"
    exit 1
}
```

> [!tip] Bonnes Pratiques Serveurs
> 1. **Documentation** : Consigner la jonction dans un registre de changements
> 2. **Sauvegarde** : Créer un snapshot/backup avant la jonction
> 3. **Test** : Valider tous les services après redémarrage
> 4. **Monitoring** : Vérifier que le serveur apparaît dans les outils de monitoring
> 5. **GPO** : Vérifier l'application des stratégies appropriées

---

## ⚙️ Gestion des Comptes Ordinateurs dans AD

Une fois joints au domaine, les comptes ordinateurs nécessitent une gestion régulière pour maintenir un annuaire propre et sécurisé.

### Visualisation des Comptes Ordinateurs

#### Via ADUC (Active Directory Users and Computers)

```
1. Démarrer → Outils d'administration → Utilisateurs et ordinateurs Active Directory
2. Ou : dsa.msc
3. Naviguer vers l'OU contenant les ordinateurs
4. Les ordinateurs apparaissent avec icône 💻
```

**Colonnes utiles à afficher :**
- Nom
- Description
- Système d'exploitation
- Dernière connexion
- Créé le
- Modifié le

```
Personnaliser l'affichage :
Clic droit sur en-têtes de colonnes → Choisir les colonnes
Sélectionner : Operating System, Operating System Version, Last Logon
```

#### Via PowerShell

```powershell
# Lister tous les ordinateurs du domaine
Get-ADComputer -Filter * | Select-Object Name, DNSHostName, Enabled

# Lister avec système d'exploitation
Get-ADComputer -Filter * -Properties OperatingSystem, OperatingSystemVersion |
    Select-Object Name, OperatingSystem, OperatingSystemVersion

# Lister par OU
Get-ADComputer -Filter * -SearchBase "OU=Workstations,DC=entreprise,DC=local" |
    Select-Object Name, DistinguishedName

# Lister avec dernière connexion
Get-ADComputer -Filter * -Properties LastLogonDate |
    Select-Object Name, LastLogonDate |
    Sort-Object LastLogonDate -Descending

# Lister uniquement les ordinateurs actifs
Get-ADComputer -Filter {Enabled -eq $true} |
    Select-Object Name, DNSHostName

# Lister uniquement les ordinateurs désactivés
Get-ADComputer -Filter {Enabled -eq $false} |
    Select-Object Name, DistinguishedName

# Compter les ordinateurs par OU
Get-ADComputer -Filter * -SearchBase "OU=Workstations,DC=entreprise,DC=local" |
    Group-Object {$_.DistinguishedName -replace '^CN=[^,]+,',''} |
    Select-Object Name, Count |
    Sort-Object Count -Descending
```

### Propriétés des Comptes Ordinateurs

```powershell
# Afficher toutes les propriétés d'un ordinateur
Get-ADComputer -Identity "PC-COMPTA-01" -Properties *

# Propriétés importantes
Get-ADComputer -Identity "PC-COMPTA-01" -Properties * |
    Select-Object Name, DNSHostName, OperatingSystem, Created, Modified, `
                  LastLogonDate, PasswordLastSet, Enabled, Description

# Afficher les groupes dont l'ordinateur est membre
Get-ADPrincipalGroupMembership -Identity "PC-COMPTA-01" |
    Select-Object Name, GroupScope, GroupCategory
```

**Propriétés clés :**

| Propriété | Description | Utilité |
|-----------|-------------|---------|
| **Name** | Nom NetBIOS | Identification courte |
| **DNSHostName** | FQDN complet | Résolution réseau |
| **OperatingSystem** | Système d'exploitation | Inventaire, ciblage GPO |
| **OperatingSystemVersion** | Version OS | Gestion des mises à jour |
| **LastLogonDate** | Dernière connexion | Détection ordinateurs obsolètes |
| **PasswordLastSet** | Dernier changement mot de passe machine | Sécurité |
| **Enabled** | Actif ou désactivé | État du compte |
| **Description** | Description libre | Documentation (utilisateur, localisation) |
| **Created** | Date de création | Audit |
| **ManagedBy** | Gestionnaire | Responsabilité |

### Modification des Propriétés

```powershell
# Modifier la description
Set-ADComputer -Identity "PC-COMPTA-01" `
               -Description "Poste comptabilité - Bureau 205 - Jean Dupont"

# Définir un gestionnaire (ManagedBy)
Set-ADComputer -Identity "PC-COMPTA-01" `
               -ManagedBy "jdupont"

# Modifier l'emplacement
Set-ADComputer -Identity "PC-COMPTA-01" `
               -Location "Paris - Bâtiment A - Étage 2"

# Ajouter des attributs personnalisés
Set-ADComputer -Identity "PC-COMPTA-01" `
               -Replace @{info="Inventaire: INV-2024-0456"}

# Renommer un ordinateur (dans AD uniquement)
Rename-ADObject -Identity "CN=PC-ANCIEN,OU=Workstations,DC=entreprise,DC=local" `
                -NewName "PC-NOUVEAU"

# Activer un compte ordinateur
Enable-ADAccount -Identity "PC-COMPTA-01"

# Désactiver un compte ordinateur
Disable-ADAccount -Identity "PC-COMPTA-01"
```

> [!warning] Renommer un Ordinateur
> **ATTENTION :** `Rename-ADObject` renomme uniquement dans AD, pas sur la machine !
> 
> Pour renommer complètement :
> 1. Sur la machine : `Rename-Computer -NewName "PC-NOUVEAU" -Restart`
> 2. OU utiliser : `Rename-Computer -NewName "PC-NOUVEAU" -DomainCredential (Get-Credential) -Restart`
> 
> Le renommage dans AD se fera automatiquement après redémarrage.

### Recherches Avancées

```powershell
# Ordinateurs Windows 10
Get-ADComputer -Filter {OperatingSystem -like "*Windows 10*"} -Properties OperatingSystem |
    Select-Object Name, OperatingSystem

# Ordinateurs Windows 11
Get-ADComputer -Filter {OperatingSystem -like "*Windows 11*"} -Properties OperatingSystem |
    Select-Object Name, OperatingSystem

# Ordinateurs Windows Server
Get-ADComputer -Filter {OperatingSystem -like "*Server*"} -Properties OperatingSystem |
    Select-Object Name, OperatingSystem

# Ordinateurs créés dans les 7 derniers jours
$date = (Get-Date).AddDays(-7)
Get-ADComputer -Filter {Created -gt $date} -Properties Created |
    Select-Object Name, Created |
    Sort-Object Created -Descending

# Ordinateurs non connectés depuis 90 jours
$date = (Get-Date).AddDays(-90)
Get-ADComputer -Filter * -Properties LastLogonDate |
    Where-Object {$_.LastLogonDate -lt $date -or $_.LastLogonDate -eq $null} |
    Select-Object Name, LastLogonDate, Enabled |
    Sort-Object LastLogonDate

# Ordinateurs désactivés
Get-ADComputer -Filter {Enabled -eq $false} -Properties LastLogonDate |
    Select-Object Name, LastLogonDate, DistinguishedName

# Ordinateurs avec mot de passe machine obsolète (> 30 jours)
$date = (Get-Date).AddDays(-30)
Get-ADComputer -Filter * -Properties PasswordLastSet |
    Where-Object {$_.PasswordLastSet -lt $date} |
    Select-Object Name, PasswordLastSet |
    Sort-Object PasswordLastSet
```

### Gestion en Masse

```powershell
# Désactiver tous les ordinateurs d'une OU
Get-ADComputer -Filter * -SearchBase "OU=Ancien-Site,OU=Workstations,DC=entreprise,DC=local" |
    Disable-ADAccount

# Déplacer tous les ordinateurs Windows 10 vers une OU spécifique
Get-ADComputer -Filter {OperatingSystem -like "*Windows 10*"} -Properties OperatingSystem |
    Move-ADObject -TargetPath "OU=Windows10,OU=Workstations,DC=entreprise,DC=local"

# Ajouter une description à tous les ordinateurs sans description
Get-ADComputer -Filter * -Properties Description |
    Where-Object {[string]::IsNullOrWhiteSpace($_.Description)} |
    ForEach-Object {
        Set-ADComputer -Identity $_ -Description "À documenter"
    }

# Supprimer tous les ordinateurs inactifs depuis 180 jours
$date = (Get-Date).AddDays(-180)
Get-ADComputer -Filter * -Properties LastLogonDate |
    Where-Object {$_.LastLogonDate -lt $date -or $_.LastLogonDate -eq $null} |
    Remove-ADComputer -Confirm:$false

# Export CSV des ordinateurs pour audit
Get-ADComputer -Filter * -Properties * |
    Select-Object Name, DNSHostName, OperatingSystem, Created, LastLogonDate, Enabled, Description |
    Export-Csv -Path "C:\Audit\Computers_$(Get-Date -Format 'yyyyMMdd').csv" -NoTypeInformation -Encoding UTF8
```

### Ajout d'Ordinateurs à des Groupes

```powershell
# Ajouter un ordinateur à un groupe
Add-ADGroupMember -Identity "GRP-Workstations-VPN" -Members "PC-COMPTA-01$"

# Note : Le $ à la fin est requis pour les comptes ordinateurs !

# Ajouter plusieurs ordinateurs
$computers = Get-ADComputer -Filter {OperatingSystem -like "*Windows 11*"}
foreach ($computer in $computers) {
    Add-ADGroupMember -Identity "GRP-Windows11-Devices" -Members $computer.Name+"$"
}

# Retirer d'un groupe
Remove-ADGroupMember -Identity "GRP-Ancien-Groupe" -Members "PC-COMPTA-01$" -Confirm:$false

# Lister les groupes dont un ordinateur est membre
Get-ADPrincipalGroupMembership -Identity "PC-COMPTA-01$" |
    Select-Object Name, GroupScope
```

> [!tip] Le Dollar $ pour les Comptes Ordinateurs
> Les comptes ordinateurs dans AD se terminent toujours par `$` dans leur SamAccountName.
> 
> **Exemples :**
> - Nom affiché : `PC-COMPTA-01`
> - SamAccountName : `PC-COMPTA-01$`
> 
> Lors de l'ajout à des groupes ou recherches, utilisez toujours le `$`.

### Création Manuelle de Comptes Ordinateurs

```powershell
# Créer un compte ordinateur pré-stagé
New-ADComputer -Name "PC-NOUVEAU-01" `
               -Path "OU=Workstations,DC=entreprise,DC=local" `
               -Description "Nouveau poste comptabilité" `
               -Enabled $true

# Créer avec gestionnaire et localisation
New-ADComputer -Name "PC-RH-10" `
               -Path "OU=RH,OU=Workstations,DC=entreprise,DC=local" `
               -Description "Poste RH - Marie Martin" `
               -Location "Paris - RH - Bureau 301" `
               -ManagedBy "mmartin" `
               -Enabled $true

# Créer et ajouter directement à un groupe
$computer = New-ADComputer -Name "PC-DEV-05" `
                           -Path "OU=Dev,OU=Workstations,DC=entreprise,DC=local" `
                           -Enabled $true `
                           -PassThru

Add-ADGroupMember -Identity "GRP-Developpeurs-Workstations" -Members $computer
```

> [!info] Comptes Pré-Stagés
> **Pourquoi créer manuellement ?**
> - Placement dans OU spécifique avant jonction
> - Application GPO dès la première connexion
> - Contrôle strict des noms (éviter auto-génération)
> - Utilisé en conjonction avec DJOIN pour déploiements
> 
> **Qui peut joindre ?**
> - Le compte qui a créé l'objet ordinateur
> - Les Domain Admins
> - Les Account Operators
> - Utilisateurs avec délégation spécifique

### Suppression de Comptes Ordinateurs

```powershell
# Supprimer un ordinateur
Remove-ADComputer -Identity "PC-OBSOLETE-01" -Confirm:$false

# Supprimer avec confirmation
Remove-ADComputer -Identity "PC-ANCIEN-02"

# Supprimer plusieurs ordinateurs
$computers = Get-ADComputer -Filter {Name -like "PC-OLD-*"}
$computers | Remove-ADComputer -Confirm:$false

# Supprimer tous les ordinateurs désactivés d'une OU
Get-ADComputer -Filter {Enabled -eq $false} `
               -SearchBase "OU=Archive,OU=Workstations,DC=entreprise,DC=local" |
    Remove-ADComputer -Confirm:$false
```

> [!warning] Prudence avec Remove-ADComputer
> - Suppression définitive (pas de corbeille AD par défaut)
> - Vérifiez toujours avant de supprimer en masse
> - Envisagez de désactiver d'abord et supprimer après période d'observation
> - Documentez les suppressions pour audit

### Monitoring et Reporting

```powershell
# Rapport complet des ordinateurs
$report = Get-ADComputer -Filter * -Properties * |
    Select-Object Name, DNSHostName, OperatingSystem, OperatingSystemVersion, `
                  Created, Modified, LastLogonDate, PasswordLastSet, Enabled, `
                  Description, Location, ManagedBy, DistinguishedName

$report | Export-Csv -Path "C:\Reports\AD-Computers-$(Get-Date -Format 'yyyyMMdd').csv" `
                     -NoTypeInformation -Encoding UTF8

# Statistiques par système d'exploitation
Get-ADComputer -Filter * -Properties OperatingSystem |
    Group-Object OperatingSystem |
    Select-Object Name, Count |
    Sort-Object Count -Descending |
    Format-Table -AutoSize

# Ordinateurs par OU
Get-ADComputer -Filter * |
    Group-Object {$_.DistinguishedName -replace '^CN=[^,]+,',''} |
    Select-Object Name, Count |
    Sort-Object Count -Descending

# Ordinateurs créés par mois (dernière année)
Get-ADComputer -Filter * -Properties Created |
    Where-Object {$_.Created -gt (Get-Date).AddYears(-1)} |
    Group-Object {$_.Created.ToString("yyyy-MM")} |
    Select-Object Name, Count |
    Sort-Object Name
```

---

## 🔄 Réinitialisation de Comptes Ordinateurs

Les comptes ordinateurs peuvent perdre leur relation de confiance avec le domaine, nécessitant une réinitialisation.

### Symptômes de Perte de Confiance

> [!warning] Signes d'un Canal Sécurisé Rompu
> **Messages d'erreur typiques :**
> - "La relation d'approbation entre cette station de travail et le domaine principal a échoué"
> - "The trust relationship between this workstation and the primary domain failed"
> - Impossible de se connecter avec compte domaine
> - `Test-ComputerSecureChannel` retourne `False`
> 
> **Causes fréquentes :**
> - Ordinateur hors ligne pendant longtemps (> 30 jours sans changement mot de passe machine)
> - Restauration d'image système ancienne
> - Clone de VM sans sysprep
> - Compte ordinateur réinitialisé dans AD pendant utilisation
> - Corruption du compte local SAM

### Diagnostic de la Relation de Confiance

```powershell
# Test rapide du canal sécurisé
Test-ComputerSecureChannel

# Test avec détails
Test-ComputerSecureChannel -Verbose

# Test avec réparation automatique (nécessite credentials domaine)
Test-ComputerSecureChannel -Repair -Credential (Get-Credential)

# Vérifier via nltest
nltest /sc_query:entreprise.local

# Afficher les informations du canal
nltest /sc_query:entreprise.local /verbose
```

**Résultats possibles :**

```powershell
# ✅ Tout va bien
True

# ❌ Problème détecté
False
```

### Méthode 1 : Réinitialisation depuis l'Ordinateur (avec accès)

Si vous pouvez encore vous connecter localement sur la machine :

```powershell
# Réinitialiser le mot de passe du compte machine
Reset-ComputerMachinePassword -Credential (Get-Credential) -Server dc01.entreprise.local

# Avec test et réparation
Test-ComputerSecureChannel -Repair -Credential (Get-Credential)

# Si succès, redémarrer
Restart-Computer
```

**Via nltest (alternative) :**

```cmd
nltest /sc_reset:entreprise.local
```

> [!info] Credentials Requis
> Vous devez fournir un compte avec droits de :
> - Domain Admin
> - Ou délégation "Reset Password" sur le compte ordinateur

### Méthode 2 : Réinitialisation depuis Active Directory

Si l'ordinateur est inaccessible ou la méthode 1 échoue :

#### Via ADUC (Interface Graphique)

```
1. Ouvrir ADUC (dsa.msc)
2. Localiser le compte ordinateur
3. Clic droit → "Reset Account"
4. Confirmer la réinitialisation
5. Sur l'ordinateur : redémarrer ou rejoindre à nouveau le domaine
```

#### Via PowerShell

```powershell
# Réinitialiser le compte ordinateur depuis AD
Reset-ADAccountPassword -Identity "PC-COMPTA-01$" -NewPassword (ConvertTo-SecureString -AsPlainText "MotDePasseTemporaire123!" -Force)

# Alternative avec Reset-ComputerMachinePassword côté serveur
# (nécessite module RSAT)
Invoke-Command -ComputerName dc01.entreprise.local -ScriptBlock {
    Reset-ComputerMachinePassword -ComputerName "PC-COMPTA-01" -Server $env:COMPUTERNAME
}
```

> [!warning] Important après Réinitialisation AD
> Après réinitialisation du compte dans AD, l'ordinateur doit :
> 
> **Option A - Rejoindre le domaine**
> ```powershell
> # Sur l'ordinateur (session admin local)
> Remove-Computer -UnjoinDomaincredential (Get-Credential) -WorkgroupName "WORKGROUP" -Restart
> 
> # Après redémarrage
> Add-Computer -DomainName "entreprise.local" -Credential (Get-Credential) -Restart
> ```
> 
> **Option B - Réinitialiser le mot de passe machine**
> ```powershell
> # Sur l'ordinateur
> Reset-ComputerMachinePassword -Credential (Get-Credential) -Server dc01.entreprise.local
> Restart-Computer
> ```

### Méthode 3 : Netdom (Outil Classique)

```cmd
REM Réinitialiser la relation de confiance
netdom resetpwd /server:dc01.entreprise.local /userd:ENTREPRISE\administrateur /passwordd:*

REM Vérifier la relation
netdom verify PC-COMPTA-01 /domain:entreprise.local
```

### Script Complet de Diagnostic et Réparation

```powershell
<#
.SYNOPSIS
    Diagnostic et réparation relation de confiance AD
.DESCRIPTION
    Teste et répare le canal sécurisé entre ordinateur et domaine
#>

function Test-AndRepairTrust {
    param(
        [Parameter(Mandatory=$false)]
        [string]$DomainController
    )
    
    Write-Host "=== Diagnostic de la relation de confiance ===" -ForegroundColor Cyan
    
    # 1. Test initial
    Write-Host "`n1. Test du canal sécurisé..." -ForegroundColor Yellow
    $initialTest = Test-ComputerSecureChannel -Verbose
    
    if ($initialTest) {
        Write-Host "✅ Canal sécurisé fonctionnel - Aucune action requise" -ForegroundColor Green
        return $true
    } else {
        Write-Host "❌ Canal sécurisé défaillant - Réparation nécessaire" -ForegroundColor Red
    }
    
    # 2. Informations système
    Write-Host "`n2. Informations système..." -ForegroundColor Yellow
    $compInfo = Get-ComputerInfo | Select-Object CsName, CsDomain, CsDomainRole
    $compInfo | Format-List
    
    # 3. Test connectivité DC
    Write-Host "`n3. Test de connectivité avec le domaine..." -ForegroundColor Yellow
    $domain = $compInfo.CsDomain
    
    try {
        $dcTest = Test-NetConnection -ComputerName $domain -Port 389 -WarningAction SilentlyContinue
        if ($dcTest.TcpTestSucceeded) {
            Write-Host "✅ Connectivité LDAP OK avec $domain" -ForegroundColor Green
        } else {
            Write-Host "❌ Impossible de contacter le domaine sur le port LDAP" -ForegroundColor Red
            Write-Host "Vérifiez le réseau et le DNS" -ForegroundColor Red
            return $false
        }
    } catch {
        Write-Host "❌ Erreur de connectivité : $_" -ForegroundColor Red
        return $false
    }
    
    # 4. Tentative de réparation
    Write-Host "`n4. Tentative de réparation automatique..." -ForegroundColor Yellow
    Write-Host "Veuillez fournir les credentials d'un compte Domain Admin :" -ForegroundColor Cyan
    
    try {
        $cred = Get-Credential -Message "Compte Domain Admin"
        
        if ($DomainController) {
            $repair = Test-ComputerSecureChannel -Repair -Credential $cred -Server $DomainController
        } else {
            $repair = Test-ComputerSecureChannel -Repair -Credential $cred
        }
        
        if ($repair) {
            Write-Host "✅ Réparation réussie !" -ForegroundColor Green
            Write-Host "`nRedémarrage recommandé pour finaliser..." -ForegroundColor Yellow
            
            $restart = Read-Host "Redémarrer maintenant ? (O/N)"
            if ($restart -eq "O") {
                Restart-Computer -Force
            }
            return $true
        } else {
            Write-Host "❌ La réparation automatique a échoué" -ForegroundColor Red
            Write-Host "`nActions recommandées :" -ForegroundColor Yellow
            Write-Host "1. Réinitialiser le compte dans Active Directory (ADUC → Reset Account)" -ForegroundColor White
            Write-Host "2. Déjoindre puis rejoindre le domaine" -ForegroundColor White
            return $false
        }
    } catch {
        Write-Host "❌ Erreur durant la réparation : $_" -ForegroundColor Red
        return $false
    }
}

# Exécution
Test-AndRepairTrust
```

### Réinitialisation via Script Distant

```powershell
# Réinitialiser plusieurs ordinateurs à distance
$computers = @("PC-COMPTA-01", "PC-RH-02", "PC-IT-03")
$credential = Get-Credential -Message "Domain Admin"

foreach ($computer in $computers) {
    Write-Host "Traitement de $computer..." -ForegroundColor Cyan
    
    try {
        Invoke-Command -ComputerName $computer -Credential $credential -ScriptBlock {
            $result = Test-ComputerSecureChannel -Repair -Credential $using:credential
            if ($result) {
                Write-Host "✅ $using:computer réparé" -ForegroundColor Green
            } else {
                Write-Host "❌ $using:computer échec" -ForegroundColor Red
            }
        }
    } catch {
        Write-Host "❌ Erreur sur $computer : $_" -ForegroundColor Red
    }
}
```

### Prévention des Problèmes de Confiance

> [!tip] Bonnes Pratiques
> **Pour éviter la perte de confiance :**
> 
> 1. **Mots de passe machine réguliers :**
>    - Par défaut : changement tous les 30 jours
>    - Ne pas désactiver cette fonctionnalité
> 
> 2. **Connexions régulières :**
>    - Ordinateurs doivent se connecter au domaine régulièrement
>    - Maximum 30 jours hors ligne recommandé
> 
> 3. **Sauvegardes/Clones :**
>    - Toujours utiliser Sysprep avant clonage de VM
>    - Ne jamais restaurer un backup système ancien (> 30 jours) sur le même nom
> 
> 4. **Surveillance :**
>    - Monitorer les ordinateurs avec mot de passe obsolète
>    - Auditer régulièrement les comptes inactifs
> 
> ```powershell
> # Détecter ordinateurs à risque (mot de passe > 25 jours)
> $date = (Get-Date).AddDays(-25)
> Get-ADComputer -Filter * -Properties PasswordLastSet |
>     Where-Object {$_.PasswordLastSet -lt $date} |
>     Select-Object Name, PasswordLastSet |
>     Sort-Object PasswordLastSet
> ```

---

## 📦 Déplacement entre OU

Le déplacement des comptes ordinateurs entre Unités Organisationnelles est une opération courante pour réorganiser l'annuaire ou appliquer différentes stratégies.

### Pourquoi Déplacer des Ordinateurs ?

> [!info] Raisons Courantes
> **Motifs de déplacement :**
> - **Application de GPO différentes** : Déplacer vers OU avec GPO spécifiques
> - **Réorganisation** : Nouvelle structure organisationnelle
> - **Localisation** : Regroupement par site géographique
> - **Type d'appareil** : Séparer laptops, desktops, serveurs
> - **Département** : Regroupement par service
> - **Quarantaine** : Isoler ordinateurs problématiques
> - **Décommissionnement** : Déplacer vers OU "Archive" avant suppression

### Méthode 1 : Via ADUC (Interface Graphique)

```
1. Ouvrir ADUC (dsa.msc)
2. Localiser l'ordinateur
3. Clic droit sur l'ordinateur → "Move" (Déplacer)
4. Sélectionner l'OU de destination
5. Cliquer OK
```

**Déplacement multiple :**
```
1. Sélectionner plusieurs ordinateurs (Ctrl + Clic)
2. Clic droit → "Move"
3. Choisir OU de destination
4. OK
```

> [!tip] Raccourci Clavier ADUC
> **Ctrl + X** : Couper
> **Ctrl + V** : Coller (dans l'OU cible)

### Méthode 2 : PowerShell (Recommandée)

```powershell
# Déplacer un ordinateur
Move-ADObject -Identity "CN=PC-COMPTA-01,OU=Ancien,DC=entreprise,DC=local" `
              -TargetPath "OU=Nouveau,OU=Workstations,DC=entreprise,DC=local"

# Alternative avec Get-ADComputer
$computer = Get-ADComputer -Identity "PC-COMPTA-01"
Move-ADObject -Identity $computer.DistinguishedName `
              -TargetPath "OU=Compta,OU=Services,DC=entreprise,DC=local"

# Déplacement avec confirmation
$computer = Get-ADComputer -Identity "PC-RH-01"
$targetOU = "OU=RH,OU=Workstations,DC=entreprise,DC=local"

Write-Host "Déplacement de $($computer.Name) vers $targetOU"
Move-ADObject -Identity $computer.DistinguishedName -TargetPath $targetOU
Write-Host "✅ Déplacement effectué" -ForegroundColor Green
```

### Déplacement en Masse

```powershell
# Déplacer tous les ordinateurs d'une OU vers une autre
$sourceOU = "OU=Ancien-Site,OU=Workstations,DC=entreprise,DC=local"
$targetOU = "OU=Archive,OU=Workstations,DC=entreprise,DC=local"

Get-ADComputer -Filter * -SearchBase $sourceOU |
    ForEach-Object {
        Write-Host "Déplacement de $($_.Name)..." -ForegroundColor Cyan
        Move-ADObject -Identity $_.DistinguishedName -TargetPath $targetOU
    }

Write-Host "✅ Déplacement de masse terminé" -ForegroundColor Green

# Déplacer basé sur système d'exploitation
# Exemple : Tous les Windows 11 vers OU dédiée
$targetOU = "OU=Windows11,OU=Workstations,DC=entreprise,DC=local"

Get-ADComputer -Filter {OperatingSystem -like "*Windows 11*"} -Properties OperatingSystem |
    ForEach-Object {
        Write-Host "Déplacement de $($_.Name) (Windows 11)..." -ForegroundColor Cyan
        Move-ADObject -Identity $_.DistinguishedName -TargetPath $targetOU
    }

# Déplacer ordinateurs inactifs vers Archive
$date = (Get-Date).AddDays(-90)
$targetOU = "OU=Inactive,OU=Archive,DC=entreprise,DC=local"

Get-ADComputer -Filter * -Properties LastLogonDate |
    Where-Object {$_.LastLogonDate -lt $date -or $_.LastLogonDate -eq $null} |
    ForEach-Object {
        Write-Host "Archive de $($_.Name) (inactif depuis $($_.LastLogonDate))..." -ForegroundColor Yellow
        Move-ADObject -Identity $_.DistinguishedName -TargetPath $targetOU
    }
```

### Déplacement avec Validation

```powershell
# Script avec validation et journalisation
function Move-ComputerToOU {
    param(
        [Parameter(Mandatory=$true)]
        [string]$ComputerName,
        
        [Parameter(Mandatory=$true)]
        [string]$TargetOU,
        
        [Parameter(Mandatory=$false)]
        [string]$LogPath = "C:\Logs\AD-Moves.log"
    )
    
    # Fonction de log
    function Write-Log {
        param([string]$Message)
        $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
        "$timestamp - $Message" | Out-File -FilePath $LogPath -Append
        Write-Host $Message
    }
    
    try {
        # Vérifier l'existence de l'ordinateur
        $computer = Get-ADComputer -Identity $ComputerName -ErrorAction Stop
        Write-Log "✓ Ordinateur trouvé : $ComputerName"
        
        # Vérifier l'existence de l'OU cible
        $ouExists = [ADSI]::Exists("LDAP://$TargetOU")
        if (-not $ouExists) {
            Write-Log "✗ OU cible inexistante : $TargetOU"
            return $false
        }
        Write-Log "✓ OU cible valide : $TargetOU"
        
        # Afficher OU actuelle
        Write-Log "OU actuelle : $($computer.DistinguishedName)"
        
        # Déplacer
        Move-ADObject -Identity $computer.DistinguishedName -TargetPath $TargetOU
        Write-Log "✅ $ComputerName déplacé vers $TargetOU"
        
        return $true
        
    } catch {
        Write-Log "❌ Erreur : $_"
        return $false
    }
}

# Utilisation
Move-ComputerToOU -ComputerName "PC-COMPTA-01" `
                  -TargetOU "OU=Compta,OU=Services,DC=entreprise,DC=local"
```

### Déplacement avec Rapport CSV

```powershell
# Déplacer ordinateurs selon CSV avec rapport
# Fichier CSV : moves.csv
# ComputerName,TargetOU
# PC-COMPTA-01,OU=Compta,OU=Services,DC=entreprise,DC=local
# PC-RH-01,OU=RH,OU=Services,DC=entreprise,DC=local

$moves = Import-Csv -Path "C:\Scripts\moves.csv"
$reportPath = "C:\Reports\Move-Report-$(Get-Date -Format 'yyyyMMdd-HHmmss').csv"
$results = @()

foreach ($move in $moves) {
    Write-Host "Traitement de $($move.ComputerName)..." -ForegroundColor Cyan
    
    try {
        $computer = Get-ADComputer -Identity $move.ComputerName -ErrorAction Stop
        $oldOU = $computer.DistinguishedName -replace "^CN=$($computer.Name),"
        
        Move-ADObject -Identity $computer.DistinguishedName -TargetPath $move.TargetOU -ErrorAction Stop
        
        $results += [PSCustomObject]@{
            ComputerName = $move.ComputerName
            OldOU = $oldOU
            NewOU = $move.TargetOU
            Status = "Success"
            ErrorMessage = ""
            Timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
        }
        
        Write-Host "✅ $($move.ComputerName) déplacé" -ForegroundColor Green
        
    } catch {
        $results += [PSCustomObject]@{
            ComputerName = $move.ComputerName
            OldOU = "N/A"
            NewOU = $move.TargetOU
            Status = "Failed"
            ErrorMessage = $_.Exception.Message
            Timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
        }
        
        Write-Host "❌ Erreur sur $($move.ComputerName): $_" -ForegroundColor Red
    }
}

# Exporter le rapport
$results | Export-Csv -Path $reportPath -NoTypeInformation -Encoding UTF8
Write-Host "`n📊 Rapport exporté : $reportPath" -ForegroundColor Cyan

# Afficher résumé
$successCount = ($results | Where-Object {$_.Status -eq "Success"}).Count
$failCount = ($results | Where-Object {$_.Status -eq "Failed"}).Count

Write-Host "`n=== RÉSUMÉ ===" -ForegroundColor Yellow
Write-Host "Succès : $successCount" -ForegroundColor Green
Write-Host "Échecs : $failCount" -ForegroundColor Red
```

### Impact du Déplacement sur les GPO

> [!warning] Application des Stratégies de Groupe
> **Important à comprendre :**
> 
> Déplacer un ordinateur change les GPO qui lui sont appliquées !
> 
> **Ce qui se passe :**
> 1. Ordinateur déplacé vers nouvelle OU
> 2. Au prochain redémarrage/gpupdate, nouvelles GPO s'appliquent
> 3. Les GPO de l'ancienne OU ne s'appliquent plus
> 
> **Impacts possibles :**
> - Paramètres de sécurité modifiés
> - Logiciels installés/désinstallés
> - Scripts de démarrage différents
> - Restrictions utilisateur changées
> - Configuration réseau modifiée

```powershell
# Forcer la mise à jour des GPO après déplacement
# Sur l'ordinateur cible
gpupdate /force

# À distance
Invoke-Command -ComputerName "PC-COMPTA-01" -ScriptBlock {
    gpupdate /force
}

# À distance avec redémarrage
Invoke-GPUpdate -Computer "PC-COMPTA-01" -Force -Boot
```

> [!tip] Tester Avant Déplacement
> **Recommandation :**
> 1. Créer une OU de test
> 2. Déplacer un seul ordinateur test
> 3. Forcer `gpupdate /force`
> 4. Vérifier avec `gpresult /r` que les bonnes GPO s'appliquent
> 5. Valider le fonctionnement
> 6. Puis déplacer les autres

### Vérification Post-Déplacement

```powershell
# Vérifier le nouvel emplacement
Get-ADComputer -Identity "PC-COMPTA-01" | Select-Object Name, DistinguishedName

# Vérifier les GPO appliquées (depuis l'ordinateur)
gpresult /r /scope:computer

# Générer rapport HTML complet
gpresult /h "C:\Temp\GPResult.html" /f

# Lister les GPO appliquées avec détails
Get-GPResultantSetOfPolicy -ReportType Html -Path "C:\Temp\RSoP.html"

# Vérifier les OU et GPO de plusieurs ordinateurs
$computers = @("PC-COMPTA-01", "PC-RH-01", "PC-IT-01")

foreach ($computer in $computers) {
    $comp = Get-ADComputer -Identity $computer
    $ou = $comp.DistinguishedName -replace "^CN=$computer,"
    
    Write-Host "$computer est dans : $ou" -ForegroundColor Cyan
}
```

### Redirection de Conteneur par Défaut

Par défaut, les nouveaux ordinateurs joints au domaine sont placés dans `CN=Computers`.

```powershell
# Voir le conteneur par défaut actuel
redircmp

# Rediriger vers une OU personnalisée
# (Nécessite droits Domain Admin)
redircmp "OU=Workstations,OU=Nouveau,DC=entreprise,DC=local"

# Vérification
redircmp
```

> [!info] Pourquoi Rediriger ?
> **Avantages :**
> - Organisation dès la jonction
> - GPO appliquées immédiatement
> - Pas besoin de déplacer manuellement après
> 
> **À savoir :**
> - Affecte TOUS les nouveaux ordinateurs joints
> - Ne déplace pas les ordinateurs existants
> - Peut être personnalisé par script de jonction avec `-OUPath`

### Script de Nettoyage et Réorganisation

```powershell
<#
.SYNOPSIS
    Réorganisation automatique des ordinateurs dans AD
.DESCRIPTION
    Déplace les ordinateurs selon des règles métier
#>

$defaultOU = "OU=Workstations,DC=entreprise,DC=local"
$win11OU = "OU=Windows11,OU=Workstations,DC=entreprise,DC=local"
$win10OU = "OU=Windows10,OU=Workstations,DC=entreprise,DC=local"
$serverOU = "OU=Servers,DC=entreprise,DC=local"
$inactiveOU = "OU=Inactive,OU=Archive,DC=entreprise,DC=local"

$inactiveDays = 90
$inactiveDate = (Get-Date).AddDays(-$inactiveDays)

Write-Host "=== Réorganisation des ordinateurs ===" -ForegroundColor Cyan

# 1. Déplacer selon OS
Write-Host "`n1. Organisation par système d'exploitation..." -ForegroundColor Yellow

# Windows 11
$win11Computers = Get-ADComputer -Filter {OperatingSystem -like "*Windows 11*"} -Properties OperatingSystem
foreach ($computer in $win11Computers) {
    if ($computer.DistinguishedName -notlike "*$win11OU*") {
        Move-ADObject -Identity $computer.DistinguishedName -TargetPath $win11OU
        Write-Host "  → $($computer.Name) vers Windows 11 OU" -ForegroundColor Cyan
    }
}

# Windows 10
$win10Computers = Get-ADComputer -Filter {OperatingSystem -like "*Windows 10*"} -Properties OperatingSystem
foreach ($computer in $win10Computers) {
    if ($computer.DistinguishedName -notlike "*$win10OU*") {
        Move-ADObject -Identity $computer.DistinguishedName -TargetPath $win10OU
        Write-Host "  → $($computer.Name) vers Windows 10 OU" -ForegroundColor Cyan
    }
}

# Serveurs
$servers = Get-ADComputer -Filter {OperatingSystem -like "*Server*"} -Properties OperatingSystem
foreach ($server in $servers) {
    if ($server.DistinguishedName -notlike "*$serverOU*") {
        Move-ADObject -Identity $server.DistinguishedName -TargetPath $serverOU
        Write-Host "  → $($server.Name) vers Servers OU" -ForegroundColor Cyan
    }
}

# 2. Archiver les inactifs
Write-Host "`n2. Archivage des ordinateurs inactifs (>$inactiveDays jours)..." -ForegroundColor Yellow

$inactiveComputers = Get-ADComputer -Filter * -Properties LastLogonDate |
    Where-Object {($_.LastLogonDate -lt $inactiveDate -or $_.LastLogonDate -eq $null) -and 
                  $_.DistinguishedName -notlike "*$inactiveOU*"}

foreach ($computer in $inactiveComputers) {
    Move-ADObject -Identity $computer.DistinguishedName -TargetPath $inactiveOU
    Write-Host "  → $($computer.Name) vers Inactive OU (dernière connexion: $($computer.LastLogonDate))" -ForegroundColor Yellow
}

Write-Host "`n✅ Réorganisation terminée" -ForegroundColor Green
```

---

## 🎯 Récapitulatif

> [!success] Points Clés à Retenir
> 
> **Jonction au Domaine :**
> - **Windows 10/11** : Interface graphique, PowerShell, ou DJOIN
> - **Windows Server** : Server Manager, PowerShell, ou Sconfig
> - Prérequis critique : DNS correctement configuré
> - Placement OU dès jonction avec `-OUPath`
> - DJOIN pour déploiements offline ou distants
> 
> **Gestion des Comptes :**
> - Comptes terminés par `$` (SamAccountName)
> - Propriétés importantes : LastLogonDate, OperatingSystem, Description
> - PowerShell pour gestion en masse
> - Audit régulier pour détecter comptes obsolètes
> 
> **Réinitialisation :**
> - Symptôme : "Trust relationship failed"
> - Test : `Test-ComputerSecureChannel`
> - Réparation : `Test-ComputerSecureChannel -Repair`
> - Alternative : Reset depuis AD puis rejointure
> - Prévention : connexions régulières, pas de clones sans sysprep
> 
> **Déplacement entre OU :**
> - Interface ADUC ou PowerShell (`Move-ADObject`)
> - Impact GPO immédiat après déplacement
> - Forcer mise à jour : `gpupdate /force`
> - Vérifier : `gpresult /r`
> - Automatisation possible pour réorganisation

---

> [!quote] Bonne Pratique Finale
> **"Une gestion proactive des comptes ordinateurs permet de maintenir un Active Directory sain et sécurisé. Automatisez les tâches répétitives, documentez les procédures, et auditez régulièrement pour détecter et corriger les anomalies avant qu'elles ne deviennent problématiques."**
