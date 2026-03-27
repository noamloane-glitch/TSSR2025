## 📋 Table des matières

- [Introduction au dépannage AD](#introduction-au-dépannage-ad)
- [Outils de diagnostic](#outils-de-diagnostic)
  - [DCDiag - Diagnostic du contrôleur de domaine](#dcdiag---diagnostic-du-contrôleur-de-domaine)
  - [Repadmin - Administration de la réplication](#repadmin---administration-de-la-réplication)
- [Problèmes de réplication](#problèmes-de-réplication)
  - [Types d'erreurs de réplication](#types-derreurs-de-réplication)
  - [Diagnostic et résolution](#diagnostic-et-résolution)
- [Résultats de stratégie de groupe](#résultats-de-stratégie-de-groupe)
  - [GPResult - Outil en ligne de commande](#gpresult---outil-en-ligne-de-commande)
  - [RSOP.msc - Console graphique](#rsopmsc---console-graphique)
- [Journaux d'événements AD](#journaux-dévénements-ad)
  - [Principaux journaux AD](#principaux-journaux-ad)
  - [Événements critiques à surveiller](#événements-critiques-à-surveiller)
- [Synchronisation horaire (W32Time)](#synchronisation-horaire-w32time)
  - [Importance de la synchronisation](#importance-de-la-synchronisation)
  - [Configuration et dépannage](#configuration-et-dépannage)

---

## Introduction au dépannage AD

> [!info] Pourquoi le dépannage AD est crucial
> Active Directory est le cœur de l'infrastructure Windows. Un problème non détecté peut entraîner :
> - Échecs d'authentification utilisateur
> - Incohérences dans les stratégies de groupe
> - Pertes de données de réplication
> - Indisponibilité des services

Le dépannage AD nécessite une approche méthodique utilisant les bons outils au bon moment.

---

## 🛠️ Outils de diagnostic

### DCDiag - Diagnostic du contrôleur de domaine

**DCDiag** (Domain Controller Diagnostic) est l'outil principal pour vérifier la santé globale d'un contrôleur de domaine.

#### Syntaxe de base

```powershell
# Test complet du DC local
dcdiag

# Test d'un DC spécifique
dcdiag /s:DC01

# Tests détaillés avec informations verbose
dcdiag /v

# Test spécifique
dcdiag /test:TestName
```

#### Tests principaux disponibles

| Test | Description | Importance |
|------|-------------|------------|
| `Connectivity` | Vérifie la connectivité réseau et DNS | ⭐⭐⭐ |
| `Replications` | Vérifie l'état de la réplication | ⭐⭐⭐ |
| `Services` | Vérifie que les services AD sont démarrés | ⭐⭐⭐ |
| `Advertising` | Vérifie que le DC s'annonce correctement | ⭐⭐⭐ |
| `FSMOCheck` | Vérifie les rôles FSMO | ⭐⭐ |
| `KnowsOfRoleHolders` | Vérifie la connaissance des détenteurs FSMO | ⭐⭐ |
| `DNS` | Vérifie l'enregistrement DNS du DC | ⭐⭐⭐ |
| `NCSecDesc` | Vérifie les descripteurs de sécurité | ⭐ |

#### Exemples d'utilisation courants

```powershell
# Test de réplication uniquement
dcdiag /test:Replications

# Test de tous les DC du domaine
dcdiag /a

# Test avec correction automatique des erreurs DNS
dcdiag /test:DNS /DnsDynamicUpdate

# Vérification des services AD essentiels
dcdiag /test:Services

# Test de la connectivité LDAP
dcdiag /test:Connectivity

# Export des résultats dans un fichier
dcdiag /v > C:\Logs\dcdiag_results.txt
```

> [!example] Exemple de sortie normale
> ```
> Starting test: Connectivity
>    * Active Directory LDAP Services Check
>    * Active Directory RPC Services Check
>    ......................... DC01 passed test Connectivity
> ```

> [!warning] Erreur courante
> Si vous voyez "LDAP bind failed with error", vérifiez :
> - Le service NTDS est démarré
> - Les ports LDAP (389, 636, 3268, 3269) ne sont pas bloqués
> - Les certificats SSL si vous utilisez LDAPS

#### Tests approfondis recommandés

```powershell
# Test DNS complet (recommandé après modifications DNS)
dcdiag /test:DNS /v /e

# Test de la topologie de réplication
dcdiag /test:KccEvent /test:Topology

# Vérification de la sécurité AD
dcdiag /test:NCSecDesc /test:ObjectsReplicated

# Test de bout en bout (complet mais long)
dcdiag /c /v /e
```

> [!tip] Astuce de diagnostic
> Utilisez `/e` pour tester tous les serveurs de l'entreprise et `/c` pour effectuer des tests complets. Combinez avec `> fichier.log` pour garder une trace.

---

### Repadmin - Administration de la réplication

**Repadmin** est l'outil spécialisé pour tout ce qui concerne la réplication AD.

#### Commandes essentielles

```powershell
# Afficher l'état de réplication de tous les partenaires
repadmin /showrepl

# Afficher l'état de réplication pour un DC spécifique
repadmin /showrepl DC01

# Forcer la réplication immédiate
repadmin /syncall

# Forcer la réplication depuis un DC source spécifique
repadmin /syncall DC01 /Aed

# Afficher les échecs de réplication dans la forêt
repadmin /replsummary

# Afficher la file d'attente de réplication
repadmin /queue
```

#### Options importantes pour /syncall

| Option | Description | Usage |
|--------|-------------|-------|
| `/A` | Tous les contextes de nommage | Recommandé |
| `/e` | Entreprise entière | Pour toute la forêt |
| `/d` | Identifie les serveurs par DN | Meilleure lisibilité |
| `/P` | Push depuis ce serveur | Force la diffusion |

```powershell
# Synchronisation complète recommandée
repadmin /syncall /AeD

# Synchronisation avec affichage détaillé
repadmin /syncall /AedP
```

#### Diagnostic des erreurs de réplication

```powershell
# Résumé des erreurs de réplication
repadmin /showrepl * /csv > replication_status.csv

# Afficher les métadonnées de réplication d'un objet
repadmin /showobjmeta "CN=User,OU=Users,DC=contoso,DC=com"

# Vérifier la connectivité entre deux DC
repadmin /bind DC01

# Afficher les liens de site de réplication
repadmin /bridgeheads

# Vérifier les cursors de réplication (USN)
repadmin /showutdvec DC01 "DC=contoso,DC=com"
```

> [!example] Comprendre les codes d'erreur de réplication
> | Code | Description | Cause probable |
> |------|-------------|----------------|
> | 8524 | "The DSA operation is unable to proceed" | Problème de connectivité réseau |
> | 8453 | "Replication access was denied" | Problème de permissions/authentification |
> | 1722 | "The RPC server is unavailable" | Pare-feu ou service RPC arrêté |
> | 1256 | "The remote system is not available" | DC éteint ou réseau coupé |
> | 8606 | "Insufficient attributes were given" | Corruption d'objet AD |

#### Commandes avancées de dépannage

```powershell
# Afficher les partitions de réplication
repadmin /showreps

# Vérifier les conflits de réplication
repadmin /showrepl * /errorsonly

# Recalculer la topologie de réplication
repadmin /kcc DC01

# Afficher les objets fantômes (lingering objects)
repadmin /removelingeringobjects DC01 "DC=contoso,DC=com" /advisory_mode

# Supprimer les objets fantômes (ATTENTION)
repadmin /removelingeringobjects DC01 "DC=contoso,DC=com" DC02 /GUID
```

> [!warning] Attention aux commandes destructives
> Les commandes comme `/removelingeringobjects` en mode exécution peuvent supprimer définitivement des objets. Toujours utiliser `/advisory_mode` d'abord pour analyser.

> [!tip] Monitoring continu
> Créez un script planifié qui exécute `repadmin /replsummary` et vous alerte si le pourcentage d'échec dépasse 5%.

---

## 🔄 Problèmes de réplication

### Types d'erreurs de réplication

#### 1. Erreurs de connectivité réseau

**Symptômes :**
- Erreur 1722 (RPC server unavailable)
- Erreur 1753 (No more endpoints available)
- Erreur 5 (Access denied)

**Diagnostic :**

```powershell
# Tester la connectivité RPC
Test-NetConnection DC01 -Port 135

# Tester les ports essentiels AD
135, 389, 636, 3268, 3269, 88, 53, 445, 49152-65535 | ForEach-Object {
    Test-NetConnection DC01 -Port $_
}

# Vérifier les règles de pare-feu
Get-NetFirewallRule -DisplayName "*Active Directory*" | Select-Object DisplayName, Enabled

# Tester l'authentification Kerberos
nltest /sc_query:contoso.com
```

#### 2. Erreurs de permissions et sécurité

**Symptômes :**
- Erreur 8453 (Replication access denied)
- Erreur 8344 (Missing replication credentials)

**Vérification :**

```powershell
# Vérifier l'appartenance au groupe "Domain Controllers"
Get-ADGroupMember "Domain Controllers"

# Vérifier les permissions de réplication
repadmin /showreps DC01

# Réinitialiser le mot de passe machine d'un DC
netdom resetpwd /s:DC02 /ud:CONTOSO\Administrateur /pd:*
```

> [!warning] Mot de passe machine DC
> Le mot de passe machine d'un DC doit être cohérent. Un désynchronisation peut bloquer la réplication. Utilisez `netdom resetpwd` en dernier recours uniquement.

#### 3. Problèmes de topologie

**Symptômes :**
- Certains DC ne répliquent pas
- Réplication lente ou incomplète
- Objets fantômes (lingering objects)

**Diagnostic :**

```powershell
# Vérifier la topologie
repadmin /istg

# Afficher les liens de réplication
Get-ADReplicationConnection -Filter *

# Forcer le Knowledge Consistency Checker (KCC)
repadmin /kcc DC01
```

#### 4. Objets USN et backups obsolètes

**Symptômes :**
- Erreur 8606 après restauration
- Objects USN rollback

**Détection :**

```powershell
# Vérifier les USN
repadmin /showutdvec DC01 "DC=contoso,DC=com"

# Détecter un USN rollback
Get-EventLog -LogName "Directory Service" -InstanceId 2095

# Supprimer les métadonnées d'un DC obsolète
ntdsutil "metadata cleanup" connections "connect to server DC01" q "select operation target" "list domains" "select domain 0" "list sites" "select site 0" "list servers in site" "select server 0" "remove selected server" q q q
```

> [!danger] USN Rollback
> Un USN rollback est critique. Il se produit après une mauvaise restauration. Le DC doit être immédiatement isolé et dépromu.

### Diagnostic et résolution

#### Méthodologie de dépannage

```powershell
# ÉTAPE 1 : Identifier le problème
repadmin /replsummary

# ÉTAPE 2 : Localiser les erreurs
repadmin /showrepl * /errorsonly

# ÉTAPE 3 : Diagnostiquer le DC
dcdiag /s:DC01 /v

# ÉTAPE 4 : Vérifier la connectivité
Test-NetConnection DC01 -Port 389

# ÉTAPE 5 : Analyser les journaux
Get-EventLog -LogName "Directory Service" -Newest 100 | Where-Object {$_.EntryType -eq "Error"}

# ÉTAPE 6 : Forcer la réplication si nécessaire
repadmin /syncall DC01 /AeD
```

#### Résolution des problèmes courants

**Problème : La réplication est bloquée**

```powershell
# Vérifier l'état de la file d'attente
repadmin /queue

# Si la file est saturée, forcer le vidage
repadmin /syncall /AedP

# Recalculer la topologie
repadmin /kcc
```

**Problème : Métadonnées orphelines**

```powershell
# Afficher les métadonnées de tous les DC
repadmin /viewlist *

# Nettoyer les métadonnées d'un DC supprimé
# Utiliser ntdsutil metadata cleanup (voir section précédente)
```

**Problème : Corruption de base AD**

```powershell
# Vérifier l'intégrité de la base
ntdsutil "activate instance ntds" "semantic database analysis" "go" "go fixup" q q

# Défragement offline (arrêter AD DS d'abord)
ntdsutil "activate instance ntds" "files" "compact to c:\temp" q q
```

> [!tip] Sauvegarde avant intervention
> Avant toute opération de maintenance lourde (cleanup, compaction), effectuez une sauvegarde système complète.

---

## 📊 Résultats de stratégie de groupe

### GPResult - Outil en ligne de commande

**GPResult** affiche les stratégies de groupe appliquées pour un utilisateur et/ou un ordinateur.

#### Syntaxe de base

```powershell
# Résultat pour l'utilisateur et l'ordinateur actuels
gpresult /r

# Résultat détaillé (Resultant Set of Policy)
gpresult /v

# Résultat ultra-détaillé
gpresult /z

# Export en HTML (recommandé)
gpresult /h C:\Reports\gpresult.html

# Export en HTML avec toutes les infos
gpresult /h C:\Reports\gpresult_full.html /f
```

#### Options avancées

```powershell
# Résultat pour un utilisateur distant
gpresult /s DC01 /user CONTOSO\jdoe /r

# Résultat avec scope spécifique
gpresult /scope:user /v
gpresult /scope:computer /v

# Format table pour PowerShell
gpresult /r /scope:computer
```

#### Informations fournies par GPResult

| Section | Information | Utilité |
|---------|-------------|---------|
| Computer Settings | GPO appliquées à l'ordinateur | Dépannage stratégies machine |
| User Settings | GPO appliquées à l'utilisateur | Dépannage stratégies utilisateur |
| Security Group Membership | Groupes de sécurité | Vérifier le filtrage de sécurité |
| Applied GPOs | Liste des GPO actives | Ordre de priorité |
| Denied GPOs | GPO bloquées | Identifier les refus |

#### Exemples pratiques

```powershell
# Générer un rapport HTML complet pour analyse
gpresult /h "C:\Reports\$(Get-Date -Format 'yyyyMMdd_HHmmss')_gpresult.html" /f

# Vérifier pourquoi une GPO n'est pas appliquée
gpresult /r /scope:user | Select-String -Pattern "NomGPO"

# Lister uniquement les GPO appliquées
gpresult /r | Select-String -Pattern "Applied Group Policy Objects"

# Diagnostic rapide de problème GPO
gpresult /h C:\temp\gpo.html && Start-Process C:\temp\gpo.html
```

> [!example] Interpréter les résultats
> ```
> Applied Group Policy Objects
> -----------------------------
>     Default Domain Policy
>     IT Security Policy
>     Desktop Wallpaper
> 
> The following GPOs were not applied because they were filtered out
> -------------------------------------------------------------------
>     Marketing GPO
>         Filtering:  Not Applied (Empty)
> ```
> Ici, "Marketing GPO" n'est pas appliqué car l'utilisateur/ordinateur n'est pas dans le bon groupe de sécurité.

> [!tip] Format HTML recommandé
> Le format HTML (`/h`) est beaucoup plus lisible et permet de naviguer facilement entre les sections. Utilisez-le systématiquement pour le dépannage.

---

### RSOP.msc - Console graphique

**RSOP** (Resultant Set of Policy) est l'interface graphique pour visualiser les GPO appliquées.

#### Lancement de RSOP

```powershell
# Lancer RSOP en mode standard
rsop.msc

# Mode planning (simulation de GPO)
# Démarrer > Exécuter > rsop.msc puis choisir "Logging Mode" ou "Planning Mode"
```

#### Deux modes d'utilisation

**1. Logging Mode (Mode journal)**
- Affiche les GPO réellement appliquées
- Données actuelles du poste
- Utilisé pour le dépannage

**2. Planning Mode (Mode planification)**
- Simule l'application de GPO
- Permet de tester avant déploiement
- Utile pour prévoir les impacts

#### Navigation dans RSOP

```
RSOP
├── Computer Configuration
│   ├── Software Settings
│   ├── Windows Settings
│   │   ├── Security Settings
│   │   └── Scripts
│   └── Administrative Templates
└── User Configuration
    ├── Software Settings
    ├── Windows Settings
    │   └── Scripts
    └── Administrative Templates
```

#### Informations disponibles

Pour chaque paramètre :
- **Winning GPO** : GPO qui définit le paramètre
- **Précédence** : Ordre d'application
- **Source** : Local, site, domaine, OU
- **Filtrage** : Groupes de sécurité, filtres WMI

> [!example] Cas d'usage typique
> Un utilisateur se plaint que le fond d'écran n'est pas celui attendu :
> 1. Lancez `rsop.msc` sur son poste
> 2. Naviguez vers User Configuration > Administrative Templates > Desktop > Desktop
> 3. Clic droit sur "Desktop Wallpaper" > Properties
> 4. Vérifiez la "Winning GPO" et l'ordre de précédence

#### Comparaison GPResult vs RSOP

| Critère | GPResult | RSOP.msc |
|---------|----------|----------|
| Interface | Ligne de commande | Graphique |
| Rapidité | ⚡ Rapide | 🐌 Plus lent |
| Export | HTML, texte | Aucun |
| Simulation | Non | Oui (Planning Mode) |
| Scriptable | ✅ Oui | ❌ Non |
| Détails techniques | ✅ Complets | Moyens |

> [!tip] Quand utiliser lequel ?
> - **GPResult** : Pour scripts, rapports automatiques, support technique rapide
> - **RSOP** : Pour analyse visuelle approfondie, formation utilisateurs, simulation

---

## 📝 Journaux d'événements AD

### Principaux journaux AD

Active Directory enregistre ses événements dans plusieurs journaux spécialisés.

#### Localisation des journaux

```powershell
# Ouvrir l'observateur d'événements
eventvwr.msc

# Via PowerShell - Lister les journaux AD
Get-WinEvent -ListLog "*Active Directory*" | Select-Object LogName, RecordCount

# Journaux principaux
Get-WinEvent -ListLog "Directory Service"
Get-WinEvent -ListLog "DFS Replication"
Get-WinEvent -ListLog "DNS Server"
```

#### Arborescence des journaux AD

```
Event Viewer
└── Applications and Services Logs
    ├── Directory Service
    ├── DFS Replication
    ├── File Replication Service
    └── DNS Server
```

#### Interrogation des journaux via PowerShell

```powershell
# Afficher les 50 derniers événements Directory Service
Get-WinEvent -LogName "Directory Service" -MaxEvents 50

# Filtrer par type d'événement (Error, Warning, Information)
Get-WinEvent -LogName "Directory Service" -MaxEvents 100 | 
    Where-Object {$_.LevelDisplayName -eq "Error"}

# Filtrer par ID d'événement
Get-WinEvent -FilterHashtable @{
    LogName = "Directory Service"
    ID = 1644
}

# Événements des dernières 24 heures
Get-WinEvent -FilterHashtable @{
    LogName = "Directory Service"
    StartTime = (Get-Date).AddDays(-1)
}

# Export des erreurs critiques
Get-WinEvent -FilterHashtable @{
    LogName = "Directory Service"
    Level = 1,2  # Critical, Error
    StartTime = (Get-Date).AddDays(-7)
} | Export-Csv C:\Logs\AD_Errors.csv -NoTypeInformation
```

> [!tip] Filtrage efficace
> Utilisez `-FilterHashtable` plutôt que `Where-Object` pour de meilleures performances sur de gros journaux.

---

### Événements critiques à surveiller

#### Événements de réplication

| Event ID | Gravité | Description | Action recommandée |
|----------|---------|-------------|-------------------|
| 1311 | ⚠️ Warning | Vérification de la cohérence échouée | Vérifier la réplication avec repadmin |
| 1388 | ❌ Error | Échec de réplication AD | Analyser la cause (réseau, DNS, auth) |
| 2042 | ⚠️ Warning | Réplication n'a pas eu lieu depuis X jours | Forcer la réplication immédiatement |
| 2087 | ❌ Error | Échec de résolution DNS pour un DC | Vérifier les enregistrements DNS SRV |
| 2095 | 🔴 Critical | Rollback détecté (USN) | Isoler le DC immédiatement |

```powershell
# Surveiller les erreurs de réplication
Get-WinEvent -FilterHashtable @{
    LogName = "Directory Service"
    ID = 1311,1388,2042,2087,2095
    StartTime = (Get-Date).AddDays(-1)
}
```

#### Événements d'authentification

| Event ID | Gravité | Description | Signification |
|----------|---------|-------------|---------------|
| 4740 | ⚠️ Warning | Compte verrouillé | Tentatives échouées répétées |
| 4625 | ⚠️ Warning | Échec d'ouverture de session | Mot de passe incorrect |
| 4768 | ℹ️ Info | Ticket Kerberos (TGT) demandé | Authentification Kerberos |
| 4776 | ℹ️ Info | Tentative de validation des informations | NTLM auth |

```powershell
# Détecter les verrouillages de compte
Get-WinEvent -FilterHashtable @{
    LogName = "Security"
    ID = 4740
    StartTime = (Get-Date).AddHours(-1)
} | Select-Object TimeCreated, Message
```

#### Événements de service AD

| Event ID | Gravité | Description | Impact |
|----------|---------|-------------|--------|
| 1000 | ❌ Error | Active Directory ne peut pas démarrer | Service NTDS non fonctionnel |
| 1168 | ❌ Error | Erreur de communication interne AD | Corruption possible |
| 1644 | ⚠️ Warning | Base AD à 90% de capacité | Planifier défragmentation |
| 2070 | ❌ Error | Échec de transfert de rôle FSMO | Vérifier la connectivité |

```powershell
# Surveiller la santé du service AD
Get-WinEvent -FilterHashtable @{
    LogName = "Directory Service"
    ID = 1000,1168,1644,2070
}
```

#### Événements DFSR (Réplication de fichiers)

| Event ID | Gravité | Description | Cause probable |
|----------|---------|-------------|----------------|
| 4012 | ⚠️ Warning | DFSR arrêté (quota dépassé) | Espace disque insuffisant |
| 2213 | ❌ Error | Échec de réplication DFSR | Problème réseau ou permissions |
| 5014 | ❌ Error | Service DFSR arrêté anormalement | Crash du service |

```powershell
# Vérifier l'état DFSR
Get-WinEvent -FilterHashtable @{
    LogName = "DFS Replication"
    Level = 1,2,3  # Critical, Error, Warning
    StartTime = (Get-Date).AddDays(-7)
}
```

#### Script de monitoring complet

```powershell
# Script de surveillance AD quotidien
$Report = @()

# Erreurs Directory Service
$DSErrors = Get-WinEvent -FilterHashtable @{
    LogName = "Directory Service"
    Level = 1,2
    StartTime = (Get-Date).AddDays(-1)
} -ErrorAction SilentlyContinue

# Erreurs de réplication critiques
$ReplErrors = Get-WinEvent -FilterHashtable @{
    LogName = "Directory Service"
    ID = 1388,2042,2095
    StartTime = (Get-Date).AddDays(-1)
} -ErrorAction SilentlyContinue

# Verrouillages de comptes
$Lockouts = Get-WinEvent -FilterHashtable @{
    LogName = "Security"
    ID = 4740
    StartTime = (Get-Date).AddDays(-1)
} -ErrorAction SilentlyContinue

# Générer le rapport
$Report += "=== RAPPORT AD - $(Get-Date) ===`n"
$Report += "Erreurs Directory Service: $($DSErrors.Count)`n"
$Report += "Erreurs de réplication: $($ReplErrors.Count)`n"
$Report += "Verrouillages de comptes: $($Lockouts.Count)`n"

# Envoyer par email si erreurs critiques
if ($ReplErrors.Count -gt 0 -or $DSErrors.Count -gt 5) {
    Send-MailMessage -To "admin@contoso.com" `
                     -From "ad-monitoring@contoso.com" `
                     -Subject "⚠️ Alerte AD" `
                     -Body ($Report -join "") `
                     -SmtpServer "smtp.contoso.com"
}
```

> [!warning] Taille des journaux
> Les journaux AD peuvent rapidement devenir volumineux. Configurez une taille maximale et une rotation automatique dans les propriétés du journal.

> [!tip] Alertes proactives
> Configurez des tâches planifiées pour surveiller les événements critiques et recevoir des alertes par email avant que les problèmes n'impactent les utilisateurs.

---

## ⏰ Synchronisation horaire (W32Time)

### Importance de la synchronisation

> [!info] Pourquoi l'heure est critique dans AD
> Une différence de temps supérieure à **5 minutes** (par défaut) entre un client et un DC provoque :
> - Échec d'authentification Kerberos (erreur "clock skew")
> - Refus des tickets Kerberos
> - Problèmes de réplication AD
> - Incohérences dans les journaux

#### Architecture de synchronisation

```
Source externe (NTP)
         ↓
    PDC Emulator (racine de la forêt)
         ↓
  Autres DC du domaine
         ↓
  Membres du domaine
```

Le **PDC Emulator** du domaine racine de la forêt est la source de temps autoritaire pour toute la forêt.

### Configuration et dépannage

#### Vérifier l'état du service W32Time

```powershell
# Vérifier que le service est démarré
Get-Service W32Time

# Démarrer le service si nécessaire
Start-Service W32Time

# Redémarrer le service
Restart-Service W32Time

# Configuration du démarrage automatique
Set-Service W32Time -StartupType Automatic
```

#### Interroger la configuration W32Time

```powershell
# Afficher la configuration actuelle
w32tm /query /configuration

# Afficher la source de temps
w32tm /query /source

# Afficher le statut de synchronisation
w32tm /query /status

# Afficher les statistiques détaillées
w32tm /query /status /verbose

# Afficher les pairs (sources NTP)
w32tm /query /peers
```

> [!example] Sortie typique de /query /source
> ```
> Free-running System Clock
> ```
> ❌ Mauvais : le serveur n'a pas de source externe
> 
> ```
> time.windows.com
> ```
> ✅ Bon : synchronisé avec une source NTP externe

#### Configuration du PDC Emulator

Le PDC Emulator doit pointer vers une source de temps externe fiable (NTP).

```powershell
# Configurer le PDC pour utiliser un serveur NTP externe
w32tm /config /manualpeerlist:"time.windows.com,0x8 pool.ntp.org,0x8" /syncfromflags:manual /reliable:yes /update

# Recharger la configuration
w32tm /config /update

# Forcer une synchronisation immédiate
w32tm /resync /rediscover

# Vérifier la synchronisation
w32tm /query /status
```

**Explications des paramètres :**

| Paramètre | Description |
|-----------|-------------|
| `/manualpeerlist` | Liste des serveurs NTP (séparés par des espaces) |
| `0x8` | Flag indiquant "utiliser comme client NTP" |
| `/syncfromflags:manual` | Synchroniser avec la liste manuelle |
| `/reliable:yes` | Marquer comme source fiable (PDC uniquement) |
| `/update` | Appliquer la configuration |

> [!tip] Sources NTP recommandées
> - **time.windows.com** (Microsoft)
> - **pool.ntp.org** (NTP Pool Project)
> - **time.google.com** (Google)
> - **time.cloudflare.com** (Cloudflare)
> 
> Utilisez plusieurs sources pour la redondance.

#### Configuration des DC non-PDC

Les autres DC doivent synchroniser avec la hiérarchie AD.

```powershell
# Configuration par défaut (hiérarchie AD)
w32tm /config /syncfromflags:domhier /update

# Redémarrer le service
Restart-Service W32Time

# Forcer la synchronisation
w32tm /resync

# Vérifier la source (doit montrer le PDC ou DC supérieur)
w32tm /query /source
```

#### Configuration des clients et serveurs membres

```powershell
# Les membres du domaine synchronisent automatiquement via GPO
# Vérifier la source (doit être un DC du domaine)
w32tm /query /source

# Forcer une re-synchronisation
w32tm /resync

# Afficher l'écart de temps
w32tm /stripchart /computer:DC01 /samples:5 /dataonly
```

#### Diagnostic des problèmes de temps

```powershell
# Tester la connectivité NTP vers une source
w32tm /stripchart /computer:time.windows.com /samples:5

# Vérifier l'écart de temps avec un DC
w32tm /stripchart /computer:DC01 /samples:10 /dataonly

# Identifier le PDC Emulator du domaine
netdom query fsmo

# Analyser les événements W32Time
Get-WinEvent -LogName System -ProviderName "Microsoft-Windows-Time-Service" -MaxEvents 50

# Debug W32Time (génère beaucoup de logs)
w32tm /debug /enable /file:C:\Logs\w32time.log /size:10000000 /entries:0-300
```

#### Résoudre les problèmes courants

**Problème : "Clock skew too great" (Kerberos)**

```powershell
# 1. Vérifier l'écart de temps
w32tm /stripchart /computer:DC01 /samples:3

# 2. Forcer une synchronisation immédiate
w32tm /resync /force

# 3. Si l'écart est important, ajuster manuellement d'abord
# (sur le client, pas sur le DC)
Set-Date (Get-Date "2025-02-10 14:30:00")
w32tm /resync

# 4. Vérifier que la synchronisation fonctionne
w32tm /query /status
```

**Problème : W32Time ne démarre pas**

```powershell
# Réenregistrer le service
w32tm /unregister
w32tm /register

# Redémarrer le service
Start-Service W32Time

# Vérifier les événements système
Get-WinEvent -LogName System | Where-Object {$_.ProviderName -like "*Time*"} | Select-Object -First 10
```

**Problème : PDC ne synchronise pas avec NTP externe**

```powershell
# Vérifier la configuration
w32tm /query /configuration

# Reconfigurer complètement
w32tm /config /manualpeerlist:"pool.ntp.org,0x8" /syncfromflags:manual /reliable:yes /update

# Arrêter et redémarrer le service
Stop-Service W32Time
Start-Service W32Time

# Forcer la synchronisation
w32tm /resync /rediscover

# Vérifier les pairs
w32tm /query /peers /verbose
```

#### Paramètres du registre W32Time

Clé de registre : `HKLM\SYSTEM\CurrentControlSet\Services\W32Time\`

| Paramètre | Chemin | Valeur par défaut | Description |
|-----------|--------|-------------------|-------------|
| MaxPosPhaseCorrection | Config | 900 (15 min) | Correction max positive |
| MaxNegPhaseCorrection | Config | 900 (15 min) | Correction max négative |
| MaxAllowedPhaseOffset | Config | 300 (5 min) | Seuil pour rejeter l'auth Kerberos |
| AnnounceFlags | Config | 10 (DC) | Annonce comme serveur de temps |

```powershell
# Modifier la tolérance de décalage (augmenter à 1 heure = 3600 secondes)
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\W32Time\Config" `
                 -Name "MaxPosPhaseCorrection" -Value 3600

Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\W32Time\Config" `
                 -Name "MaxNegPhaseCorrection" -Value 3600

# Redémarrer le service pour appliquer
Restart-Service W32Time
```

> [!warning] Modification des valeurs par défaut
> Augmenter `MaxPosPhaseCorrection` et `MaxNegPhaseCorrection` peut masquer des problèmes plus profonds. Ne le faites qu'en cas de besoin temporaire.

#### Monitoring W32Time

```powershell
# Script de vérification de la synchronisation
$DCs = Get-ADDomainController -Filter *

foreach ($DC in $DCs) {
    Write-Host "Vérification de $($DC.HostName)..." -ForegroundColor Cyan
    
    # Tester la connectivité
    if (Test-Connection -ComputerName $DC.HostName -Count 1 -Quiet) {
        
        # Récupérer la source de temps
        $Source = w32tm /query /computer:$DC.HostName /source
        
        # Récupérer le statut
        $Status = w32tm /query /computer:$DC.HostName /status
        
        Write-Host "  Source: $Source" -ForegroundColor Green
        Write-Host "  Status: $($Status | Select-String 'Stratum|Last')" 
    } else {
        Write-Host "  ❌ Non joignable" -ForegroundColor Red
    }
}
```

> [!tip] GPO pour la configuration du temps
> Créez une GPO pour standardiser la configuration W32Time sur tous les DC :
> - Computer Configuration > Policies > Administrative Templates > System > Windows Time Service
> - Configurez "Configure Windows NTP Client" et "Enable Windows NTP Server"

---

## 🎯 Bonnes pratiques de dépannage

> [!tip] Méthodologie générale
> 1. **Collecter les informations** : dcdiag, repadmin, journaux
> 2. **Isoler le problème** : un DC ? tous les DC ? un site ?
> 3. **Identifier la cause racine** : réseau, DNS, permissions, corruption ?
> 4. **Tester la solution** : dans un environnement de test si possible
> 5. **Documenter** : garder une trace pour les incidents futurs
> 6. **Surveiller** : vérifier que le problème ne revient pas

> [!warning] Pièges courants
> - Ne jamais restaurer un DC depuis une sauvegarde ancienne (risque USN rollback)
> - Ne pas désactiver un pare-feu sans comprendre les flux nécessaires
> - Ne pas supprimer manuellement des objets AD pour "réparer" la réplication
> - Ne pas modifier le registre W32Time sur les DC non-PDC sans raison valable

### Checklist de dépannage AD

```
☐ 1. Vérifier les services AD (dcdiag /test:Services)
☐ 2. Vérifier la réplication (repadmin /replsummary)
☐ 3. Vérifier DNS (dcdiag /test:DNS /v)
☐ 4. Vérifier la connectivité réseau (Test-NetConnection)
☐ 5. Vérifier la synchronisation horaire (w32tm /query /status)
☐ 6. Analyser les journaux d'événements
☐ 7. Vérifier les GPO appliquées (gpresult /h)
☐ 8. Tester l'authentification (nltest /sc_query)
☐ 9. Vérifier les rôles FSMO (netdom query fsmo)
☐ 10. Documenter la solution appliquée
```

### Outils essentiels à connaître

| Outil | Utilisation principale | Commande rapide |
|-------|------------------------|----------------|
| **dcdiag** | Santé globale du DC | `dcdiag /v` |
| **repadmin** | Réplication AD | `repadmin /replsummary` |
| **gpresult** | Stratégies de groupe | `gpresult /h report.html` |
| **w32tm** | Synchronisation horaire | `w32tm /query /status` |
| **nltest** | Authentification/trust | `nltest /sc_query:domain` |
| **netdom** | Gestion domaine | `netdom query fsmo` |
| **Get-WinEvent** | Journaux d'événements | `Get-WinEvent -LogName "Directory Service"` |

---

## 📚 Résumé des commandes essentielles

```powershell
# === DIAGNOSTIC GÉNÉRAL ===
dcdiag /v                          # Test complet du DC
dcdiag /test:Replications          # Test réplication uniquement
dcdiag /test:DNS /v                # Test DNS complet

# === RÉPLICATION ===
repadmin /replsummary              # Résumé réplication
repadmin /showrepl                 # Détails réplication
repadmin /syncall /AeD             # Forcer réplication
repadmin /showrepl * /errorsonly   # Erreurs uniquement

# === GPO ===
gpresult /r                        # Résumé GPO
gpresult /h C:\report.html         # Rapport HTML
rsop.msc                           # Console graphique

# === JOURNAUX ===
Get-WinEvent -LogName "Directory Service" -MaxEvents 50
Get-WinEvent -FilterHashtable @{LogName="Directory Service"; Level=1,2}

# === TEMPS ===
w32tm /query /status               # Statut synchronisation
w32tm /query /source               # Source de temps
w32tm /resync                      # Forcer synchronisation
w32tm /stripchart /computer:DC01   # Tester écart de temps

# === CONNECTIVITÉ ===
Test-NetConnection DC01 -Port 389  # Test LDAP
nltest /sc_query:contoso.com       # Test secure channel
```

---

> [!success] Fin du cours
> Vous maîtrisez maintenant les outils et techniques essentiels pour diagnostiquer et résoudre les problèmes Active Directory. Le dépannage AD est une compétence qui s'affine avec la pratique et l'expérience.