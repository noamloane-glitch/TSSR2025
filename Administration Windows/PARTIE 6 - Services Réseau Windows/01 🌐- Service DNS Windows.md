## 📋 Table des matières

1. [Introduction au DNS](#introduction-au-dns)
2. [Rôle DNS Server](#rôle-dns-server)
3. [Zones DNS](#zones-dns)
4. [Enregistrements DNS](#enregistrements-dns)
5. [DNS et Active Directory](#dns-et-active-directory)
6. [Transferts de zone](#transferts-de-zone)
7. [Redirecteurs DNS](#redirecteurs-dns)
8. [Dépannage DNS](#dépannage-dns)

---

## 🎯 Introduction au DNS

Le **Domain Name System (DNS)** est un service réseau fondamental qui traduit les noms de domaine lisibles par l'homme (comme `www.exemple.com`) en adresses IP compréhensibles par les machines (comme `192.168.1.10`).

> [!info] Pourquoi le DNS est-il crucial ?
> - Permet aux utilisateurs d'accéder aux ressources réseau sans mémoriser des adresses IP
> - Indispensable au fonctionnement d'Active Directory
> - Facilite la gestion et la mobilité des ressources réseau
> - Point central de la résolution de noms dans un environnement Windows

---

## 🖥️ Rôle DNS Server

### Installation du rôle DNS

Le rôle DNS Server transforme un serveur Windows en serveur DNS capable de résoudre des requêtes de noms.

**Installation via PowerShell :**

```powershell
# Installation du rôle DNS Server
Install-WindowsFeature -Name DNS -IncludeManagementTools

# Vérification de l'installation
Get-WindowsFeature -Name DNS
```

**Installation via Server Manager :**
1. Ouvrir Server Manager
2. Cliquer sur "Add roles and features"
3. Sélectionner "DNS Server"
4. Suivre l'assistant d'installation

> [!tip] Bonne pratique
> Lors de la promotion d'un contrôleur de domaine, le rôle DNS est automatiquement proposé à l'installation. Il est fortement recommandé de l'accepter car Active Directory nécessite DNS pour fonctionner correctement.

### Configuration de base

```powershell
# Définir les paramètres d'écoute du serveur DNS
Set-DnsServerSetting -ListenAddresses "192.168.1.10"

# Configurer le serveur pour accepter les requêtes récursives
Set-DnsServerRecursion -Enable $true

# Activer la journalisation des requêtes DNS (pour le dépannage)
Set-DnsServerDiagnostics -Queries $true -Answers $true
```

### Propriétés du serveur DNS

| Propriété | Description | Utilisation |
|-----------|-------------|-------------|
| **Interfaces** | Définit sur quelles interfaces réseau le serveur écoute | Serveur multi-homed (plusieurs cartes réseau) |
| **Redirecteurs** | Serveurs DNS vers lesquels transférer les requêtes non résolues | Résolution de noms externes |
| **Racine** | Indications de racine pour la résolution Internet | Résolution de noms publics |
| **Mise en cache** | Stockage temporaire des réponses DNS | Optimisation des performances |

> [!warning] Attention aux conflits
> Si votre serveur DNS possède plusieurs interfaces réseau, assurez-vous de configurer correctement les interfaces d'écoute pour éviter les conflits ou les problèmes de sécurité.

---

## 📁 Zones DNS

Une **zone DNS** est une portion de l'espace de noms DNS dont un serveur DNS particulier a la responsabilité.

### Types de zones

#### 🔵 Zone Primaire (Primary Zone)

La zone primaire contient la **copie maître** des enregistrements DNS. Toutes les modifications sont effectuées sur cette zone.

**Création d'une zone primaire :**

```powershell
# Créer une zone de recherche directe primaire
Add-DnsServerPrimaryZone -Name "entreprise.local" -ZoneFile "entreprise.local.dns"

# Créer une zone de recherche inversée primaire
Add-DnsServerPrimaryZone -NetworkId "192.168.1.0/24" -ZoneFile "1.168.192.in-addr.arpa.dns"
```

**Caractéristiques :**
- Lecture et écriture autorisées
- Source autoritaire des données
- Stockée dans un fichier `.dns` ou intégrée à Active Directory

> [!info] Quand utiliser une zone primaire ?
> - Lorsque vous gérez un nouveau domaine DNS
> - Pour un domaine dont vous êtes l'autorité unique
> - Dans un environnement Active Directory (préférer alors une zone intégrée AD)

#### 🟢 Zone Secondaire (Secondary Zone)

La zone secondaire est une **copie en lecture seule** d'une zone primaire, obtenue par transfert de zone.

**Création d'une zone secondaire :**

```powershell
# Créer une zone secondaire depuis un serveur primaire
Add-DnsServerSecondaryZone -Name "entreprise.local" `
    -ZoneFile "entreprise.local.dns" `
    -MasterServers "192.168.1.10"
```

**Caractéristiques :**
- Lecture seule
- Obtenue par transfert de zone depuis le primaire
- Fournit redondance et répartition de charge
- Mise à jour automatique selon le SOA (Serial, Refresh, Retry)

> [!tip] Bonnes pratiques
> - Déployez au minimum deux serveurs DNS (un primaire et un secondaire) pour la haute disponibilité
> - Placez les serveurs secondaires sur des sites géographiques différents
> - Surveillez les journaux de transferts de zone

#### 🟡 Zone Stub

Une zone stub contient uniquement les enregistrements **NS (Name Server)** et **SOA** d'une zone déléguée.

**Création d'une zone stub :**

```powershell
# Créer une zone stub
Add-DnsServerStubZone -Name "filiale.entreprise.local" `
    -MasterServers "192.168.2.10" `
    -ZoneFile "filiale.entreprise.local.dns"
```

**Caractéristiques :**
- Contient uniquement les références aux serveurs DNS autoritaires
- Facilite la délégation de sous-domaines
- Mise à jour automatique de la liste des serveurs de noms

> [!example] Cas d'usage typique
> Vous gérez `entreprise.local` et vous voulez déléguer `paris.entreprise.local` à un autre serveur DNS. Une zone stub vous permet de maintenir automatiquement les références vers les serveurs DNS de Paris sans gérer tous leurs enregistrements.

### Zones de recherche

#### Recherche directe (Forward Lookup Zone)

Résout un **nom → adresse IP**.

```powershell
# Exemple : www.entreprise.local → 192.168.1.50
Add-DnsServerResourceRecordA -Name "www" `
    -ZoneName "entreprise.local" `
    -IPv4Address "192.168.1.50"
```

#### Recherche inversée (Reverse Lookup Zone)

Résout une **adresse IP → nom**.

```powershell
# Créer une zone de recherche inversée
Add-DnsServerPrimaryZone -NetworkId "192.168.1.0/24" `
    -ReplicationScope "Forest"

# Ajouter un enregistrement PTR
Add-DnsServerResourceRecordPtr -Name "50" `
    -ZoneName "1.168.192.in-addr.arpa" `
    -PtrDomainName "www.entreprise.local"
```

> [!warning] Importance de la recherche inversée
> - Requise par certains services (Exchange, serveurs de messagerie)
> - Utilisée pour la vérification anti-spam
> - Nécessaire pour certaines authentifications réseau

### Gestion des zones

```powershell
# Lister toutes les zones
Get-DnsServerZone

# Obtenir les détails d'une zone spécifique
Get-DnsServerZone -Name "entreprise.local"

# Supprimer une zone
Remove-DnsServerZone -Name "test.local" -Force

# Suspendre une zone (la rendre indisponible temporairement)
Suspend-DnsServerZone -Name "entreprise.local"

# Reprendre une zone suspendue
Resume-DnsServerZone -Name "entreprise.local"
```

---

## 📝 Enregistrements DNS

Les enregistrements DNS stockent les informations qui permettent la résolution de noms.

### 🔹 Enregistrement A (Address)

Associe un **nom d'hôte IPv4** à une **adresse IPv4**.

```powershell
# Créer un enregistrement A
Add-DnsServerResourceRecordA -Name "serveur1" `
    -ZoneName "entreprise.local" `
    -IPv4Address "192.168.1.20" `
    -TimeToLive (New-TimeSpan -Hours 1)

# Créer avec plusieurs adresses IP (round-robin)
Add-DnsServerResourceRecordA -Name "web" `
    -ZoneName "entreprise.local" `
    -IPv4Address "192.168.1.30"
    
Add-DnsServerResourceRecordA -Name "web" `
    -ZoneName "entreprise.local" `
    -IPv4Address "192.168.1.31"
```

> [!info] Utilisation courante
> - Serveurs web
> - Serveurs d'applications
> - Stations de travail
> - Tout périphérique réseau avec IPv4

### 🔹 Enregistrement AAAA (IPv6 Address)

Associe un **nom d'hôte IPv6** à une **adresse IPv6**.

```powershell
# Créer un enregistrement AAAA
Add-DnsServerResourceRecordAAAA -Name "serveur1" `
    -ZoneName "entreprise.local" `
    -IPv6Address "2001:db8:85a3::8a2e:370:7334"
```

> [!tip] IPv6 dans l'entreprise
> Même si votre réseau est principalement IPv4, préparez-vous pour IPv6. De nombreux systèmes modernes privilégient IPv6 lorsqu'il est disponible.

### 🔹 Enregistrement CNAME (Canonical Name)

Crée un **alias** pointant vers un autre nom DNS.

```powershell
# Créer un alias CNAME
Add-DnsServerResourceRecordCName -Name "www" `
    -ZoneName "entreprise.local" `
    -HostNameAlias "serveur-web.entreprise.local"

# Exemple : intranet pointant vers le même serveur que www
Add-DnsServerResourceRecordCName -Name "intranet" `
    -ZoneName "entreprise.local" `
    -HostNameAlias "www.entreprise.local"
```

**Caractéristiques :**
- Ne pointe jamais vers une adresse IP, uniquement vers un nom
- Un CNAME ne peut coexister avec d'autres types d'enregistrements pour le même nom
- Résolution en deux temps : alias → nom canonique → adresse IP

> [!warning] Restrictions importantes
> - Ne créez JAMAIS de CNAME à la racine d'une zone (apex)
> - Un CNAME ne peut pas pointer vers un autre CNAME (chaînage interdit)
> - Ne mélangez pas CNAME avec A, MX, ou autres types pour le même nom

> [!example] Cas d'usage pratiques
> - **www** → serveur-web (si le serveur change, modifier un seul enregistrement A)
> - **ftp** → serveur-fichiers
> - **mail** → serveur-exchange

### 🔹 Enregistrement MX (Mail Exchanger)

Spécifie les **serveurs de messagerie** pour un domaine.

```powershell
# Créer un enregistrement MX
Add-DnsServerResourceRecordMX -Name "." `
    -ZoneName "entreprise.local" `
    -MailExchange "mail1.entreprise.local" `
    -Preference 10

# Ajouter un MX secondaire avec priorité plus basse
Add-DnsServerResourceRecordMX -Name "." `
    -ZoneName "entreprise.local" `
    -MailExchange "mail2.entreprise.local" `
    -Preference 20
```

**Paramètres :**
- **Name** : "." pour la racine du domaine, ou un sous-domaine spécifique
- **Preference** : Priorité (plus le chiffre est bas, plus la priorité est haute)
- **MailExchange** : FQDN du serveur de messagerie

> [!info] Fonctionnement des priorités
> - Les serveurs SMTP tentent d'abord le MX avec la valeur la plus **basse**
> - Si celui-ci est indisponible, ils essaient le suivant
> - Plusieurs MX avec la même priorité = répartition de charge

> [!warning] Attention
> Un enregistrement MX doit TOUJOURS pointer vers un nom (A ou AAAA), jamais vers une adresse IP directement ou un CNAME.

### 🔹 Enregistrement SRV (Service)

Localise des **services spécifiques** dans le domaine (crucial pour Active Directory).

```powershell
# Format général d'un SRV : _service._protocole.domaine
# Créer un enregistrement SRV pour LDAP
Add-DnsServerResourceRecord -ZoneName "entreprise.local" `
    -Srv -Name "_ldap._tcp" `
    -DomainName "dc1.entreprise.local" `
    -Priority 0 `
    -Weight 100 `
    -Port 389

# Enregistrement SRV pour Kerberos
Add-DnsServerResourceRecord -ZoneName "entreprise.local" `
    -Srv -Name "_kerberos._tcp" `
    -DomainName "dc1.entreprise.local" `
    -Priority 0 `
    -Weight 100 `
    -Port 88
```

**Structure d'un enregistrement SRV :**
```
_service._protocole.domaine  TTL  IN  SRV  priorité  poids  port  cible
```

**Paramètres :**
- **Priority** : Priorité du serveur (0 = plus haute)
- **Weight** : Poids pour la répartition de charge entre serveurs de même priorité
- **Port** : Numéro de port du service
- **Target** : FQDN du serveur fournissant le service

> [!info] Services courants utilisant SRV
> - **_ldap._tcp** : Service LDAP (port 389)
> - **_kerberos._tcp** : Authentification Kerberos (port 88)
> - **_gc._tcp** : Global Catalog (port 3268)
> - **_kerberos._udp** : Kerberos sur UDP
> - **_sip._tcp** : SIP (VoIP, Skype for Business)

> [!warning] Active Directory et SRV
> Les enregistrements SRV sont créés automatiquement lors de la promotion d'un contrôleur de domaine. Si ces enregistrements sont manquants ou incorrects, Active Directory ne fonctionnera pas correctement.

### 🔹 Enregistrement PTR (Pointer)

Utilisé dans les zones de recherche inversée pour résoudre une **adresse IP → nom**.

```powershell
# Créer un enregistrement PTR
Add-DnsServerResourceRecordPtr -Name "20" `
    -ZoneName "1.168.192.in-addr.arpa" `
    -PtrDomainName "serveur1.entreprise.local"

# Vérifier la résolution inversée
Resolve-DnsName -Name "192.168.1.20" -Type PTR
```

> [!tip] Automatisation
> Activez la création automatique des PTR lors de l'ajout d'enregistrements A :
> - Dans les propriétés de la zone de recherche directe
> - Cocher "Update associated pointer (PTR) record"

### 🔹 Autres types d'enregistrements

| Type | Description | Utilisation |
|------|-------------|-------------|
| **NS** | Name Server | Définit les serveurs DNS autoritaires pour une zone |
| **SOA** | Start of Authority | Informations de zone (numéro de série, TTL, etc.) |
| **TXT** | Text | Informations textuelles (SPF, DKIM, vérification domaine) |
| **CAA** | Certification Authority Authorization | Autorise les CA à émettre des certificats |

```powershell
# Enregistrement TXT (exemple SPF)
Add-DnsServerResourceRecord -ZoneName "entreprise.local" `
    -Txt -Name "@" `
    -DescriptiveText "v=spf1 mx a:mail.entreprise.local -all"

# Enregistrement NS (délégation)
Add-DnsServerResourceRecord -ZoneName "entreprise.local" `
    -NS -Name "filiale" `
    -NameServer "ns1.filiale.entreprise.local"
```

### Gestion des enregistrements

```powershell
# Lister tous les enregistrements d'une zone
Get-DnsServerResourceRecord -ZoneName "entreprise.local"

# Filtrer par type
Get-DnsServerResourceRecord -ZoneName "entreprise.local" -RRType A

# Supprimer un enregistrement A
Remove-DnsServerResourceRecord -ZoneName "entreprise.local" `
    -Name "serveur1" `
    -RRType A `
    -Force

# Modifier le TTL d'un enregistrement
$record = Get-DnsServerResourceRecord -Name "www" `
    -ZoneName "entreprise.local" -RRType A
$newRecord = $record.Clone()
$newRecord.TimeToLive = New-TimeSpan -Hours 2
Set-DnsServerResourceRecord -NewInputObject $newRecord `
    -OldInputObject $record `
    -ZoneName "entreprise.local"
```

> [!tip] Astuce TTL
> - **TTL court** (ex: 5 minutes) : pour des serveurs susceptibles de changer
> - **TTL long** (ex: 24 heures) : pour des serveurs stables
> - Diminuez le TTL AVANT un changement prévu, puis rallongez-le après

---

## 🔐 DNS et Active Directory

L'intégration entre DNS et Active Directory est **fondamentale** au bon fonctionnement d'un domaine Windows.

### Zones intégrées Active Directory

Une zone intégrée AD stocke ses données dans Active Directory au lieu d'un fichier texte.

**Avantages des zones intégrées AD :**

| Avantage | Description |
|----------|-------------|
| **Réplication multimaître** | Tous les DC peuvent modifier la zone (plus de distinction primaire/secondaire) |
| **Réplication sécurisée** | Utilise la réplication AD (chiffrée et compressée) |
| **Sauvegardes incluses** | Sauvegardé avec AD (pas de fichiers séparés) |
| **Mises à jour dynamiques sécurisées** | Seuls les ordinateurs autorisés peuvent s'enregistrer |

**Création d'une zone intégrée AD :**

```powershell
# Zone intégrée avec réplication sur tous les DC du domaine
Add-DnsServerPrimaryZone -Name "entreprise.local" `
    -ReplicationScope "Domain" `
    -DynamicUpdate "Secure"

# Zone intégrée avec réplication sur tous les DC de la forêt
Add-DnsServerPrimaryZone -Name "entreprise.local" `
    -ReplicationScope "Forest" `
    -DynamicUpdate "Secure"

# Zone intégrée avec réplication personnalisée
Add-DnsServerPrimaryZone -Name "entreprise.local" `
    -ReplicationScope "Custom" `
    -DirectoryPartitionName "DC=CustomPartition,DC=entreprise,DC=local" `
    -DynamicUpdate "Secure"
```

**Portées de réplication :**

| Portée | Description | Cas d'usage |
|--------|-------------|-------------|
| **Forest** | Tous les DC de la forêt | Zones nécessaires partout (ex: _msdcs) |
| **Domain** | Tous les DC du domaine | Zones standard du domaine |
| **Custom** | Partition d'application personnalisée | Réplication granulaire avancée |

> [!warning] Prérequis
> Pour créer des zones intégrées AD, le serveur DNS doit être installé sur un contrôleur de domaine.

### Mises à jour dynamiques

Les mises à jour dynamiques permettent aux clients de s'enregistrer automatiquement dans le DNS.

```powershell
# Activer les mises à jour dynamiques sécurisées
Set-DnsServerPrimaryZone -Name "entreprise.local" `
    -DynamicUpdate "Secure"

# Désactiver les mises à jour dynamiques
Set-DnsServerPrimaryZone -Name "entreprise.local" `
    -DynamicUpdate "None"

# Autoriser les mises à jour non sécurisées (déconseillé)
Set-DnsServerPrimaryZone -Name "entreprise.local" `
    -DynamicUpdate "NonsecureAndSecure"
```

**Types de mises à jour :**

- **None** : Aucune mise à jour automatique (gestion manuelle uniquement)
- **Secure** : Seuls les membres du domaine peuvent s'enregistrer
- **NonsecureAndSecure** : Tout client peut s'enregistrer (risque de sécurité)

> [!tip] Bonnes pratiques
> - Utilisez **toujours** "Secure" dans un environnement Active Directory
> - Réservez "NonsecureAndSecure" aux zones de test ou réseaux non-AD
> - Surveillez les enregistrements obsolètes (scavenging)

### Vieillissement et nettoyage (Scavenging)

Le scavenging supprime automatiquement les enregistrements DNS obsolètes.

```powershell
# Activer le scavenging sur le serveur
Set-DnsServerScavenging -ScavengingState $true `
    -ScavengingInterval (New-TimeSpan -Days 7) `
    -RefreshInterval (New-TimeSpan -Days 7) `
    -NoRefreshInterval (New-TimeSpan -Days 7)

# Activer le scavenging sur une zone
Set-DnsServerZoneAging -Name "entreprise.local" `
    -Aging $true `
    -RefreshInterval (New-TimeSpan -Days 7) `
    -NoRefreshInterval (New-TimeSpan -Days 7)

# Lancer un nettoyage manuel
Start-DnsServerScavenging -Force
```

**Paramètres du scavenging :**

- **NoRefreshInterval** : Période pendant laquelle un enregistrement ne peut pas être rafraîchi
- **RefreshInterval** : Période pendant laquelle un enregistrement peut être rafraîchi
- **ScavengingInterval** : Fréquence d'exécution du nettoyage

> [!warning] Configuration prudente
> - Testez d'abord sur une zone de test
> - Des intervalles trop courts peuvent supprimer des enregistrements légitimes
> - Recommandation : 7 jours pour chaque intervalle

### Enregistrements Active Directory critiques

Certains enregistrements SRV sont essentiels au fonctionnement d'AD :

```powershell
# Vérifier la présence des enregistrements critiques
nslookup -type=SRV _ldap._tcp.entreprise.local
nslookup -type=SRV _kerberos._tcp.entreprise.local
nslookup -type=SRV _gc._tcp.entreprise.local

# Forcer le réenregistrement des enregistrements AD
ipconfig /registerdns
```

> [!info] Zone _msdcs
> Active Directory crée automatiquement une zone `_msdcs.domaine.local` contenant des enregistrements SRV critiques. Cette zone doit être répliquée sur toute la forêt.

---

## 🔄 Transferts de zone

Les transferts de zone copient les données DNS d'un serveur vers un autre.

### Types de transferts

| Type | Description | Utilisation |
|------|-------------|-------------|
| **AXFR** | Transfert complet de zone | Premier transfert ou après modification majeure |
| **IXFR** | Transfert incrémental | Transferts réguliers (seules les modifications) |

### Configuration des transferts

```powershell
# Autoriser les transferts vers des serveurs spécifiques
Set-DnsServerPrimaryZone -Name "entreprise.local" `
    -SecureSecondaries "TransferToSecureServers" `
    -SecondaryServers "192.168.1.11","192.168.1.12"

# Autoriser les transferts vers n'importe quel serveur (déconseillé)
Set-DnsServerPrimaryZone -Name "entreprise.local" `
    -SecureSecondaries "TransferAnyServer"

# Désactiver complètement les transferts
Set-DnsServerPrimaryZone -Name "entreprise.local" `
    -SecureSecondaries "NoTransfer"

# Configurer les notifications de zone
Set-DnsServerPrimaryZone -Name "entreprise.local" `
    -Notify "NotifyServers" `
    -NotifyServers "192.168.1.11","192.168.1.12"
```

### Paramètres SOA influençant les transferts

L'enregistrement SOA contient des paramètres qui contrôlent les transferts :

```powershell
# Consulter les paramètres SOA
Get-DnsServerResourceRecord -ZoneName "entreprise.local" -RRType SOA

# Modifier les paramètres SOA
Set-DnsServerResourceRecord -ZoneName "entreprise.local" `
    -InputObject (Get-DnsServerResourceRecord -ZoneName "entreprise.local" -RRType SOA) `
    -NewInputObject $nouvelSOA
```

**Champs importants du SOA :**

- **Serial** : Numéro de version de la zone (incrémenté à chaque modification)
- **Refresh** : Intervalle de vérification par les secondaires (ex: 900 secondes)
- **Retry** : Intervalle de nouvelle tentative en cas d'échec (ex: 600 secondes)
- **Expire** : Durée avant expiration si le primaire est inaccessible (ex: 86400 secondes)
- **Minimum TTL** : TTL par défaut pour les enregistrements

> [!tip] Optimisation
> - **Refresh court** (15 min) pour des zones changeant fréquemment
> - **Refresh long** (plusieurs heures) pour des zones stables
> - **Expire** suffisamment long pour couvrir une panne prolongée

### Notifications de zone

```powershell
# Activer les notifications pour avertir les secondaires des changements
Set-DnsServerPrimaryZone -Name "entreprise.local" `
    -Notify "NotifyServers" `
    -NotifyServers "192.168.1.11","192.168.1.12"

# Désactiver les notifications
Set-DnsServerPrimaryZone -Name "entreprise.local" `
    -Notify "NoNotify"
```

> [!info] Fonctionnement des notifications
> Lorsqu'une modification est apportée à la zone primaire, le serveur envoie immédiatement une notification aux serveurs secondaires, qui peuvent alors initier un transfert de zone sans attendre l'intervalle "Refresh".

### Sécurisation des transferts

```powershell
# Limiter les transferts à des serveurs spécifiques via ACL
$acl = Get-DnsServerZoneTransferPolicy -ZoneName "entreprise.local"
Add-DnsServerZoneTransferPolicy -ZoneName "entreprise.local" `
    -Name "TransfertSecurise" `
    -Action Allow `
    -ServerInterfaceIP "192.168.1.11","192.168.1.12"
```

> [!warning] Sécurité critique
> - Ne permettez JAMAIS les transferts vers "Any server" en production
> - Limitez toujours aux serveurs secondaires de confiance
> - Utilisez des règles de pare-feu supplémentaires (port TCP 53)
> - Considérez l'utilisation de TSIG pour signer les transferts

---

## ➡️ Redirecteurs DNS

Les redirecteurs (forwarders) sont des serveurs DNS vers lesquels votre serveur transfère les requêtes qu'il ne peut pas résoudre lui-même.

### Configuration des redirecteurs

```powershell
# Configurer des redirecteurs globaux
Set-DnsServerForwarder -IPAddress "8.8.8.8","8.8.4.4" -EnableReordering $true

# Ajouter un redirecteur
Add-DnsServerForwarder -IPAddress "1.1.1.1"

# Supprimer un redirecteur
Remove-DnsServerForwarder -IPAddress "8.8.4.4" -Force

# Consulter les redirecteurs configurés
Get-DnsServerForwarder
```

### Redirecteurs conditionnels

Les redirecteurs conditionnels transfèrent les requêtes pour un domaine spécifique vers des serveurs DNS dédiés.

```powershell
# Créer un redirecteur conditionnel
Add-DnsServerConditionalForwarderZone -Name "partenaire.com" `
    -MasterServers "203.0.113.10","203.0.113.11" `
    -ForwarderTimeout 5

# Redirecteur conditionnel intégré AD
Add-DnsServerConditionalForwarderZone -Name "filiale.entreprise.local" `
    -MasterServers "192.168.2.10" `
    -ReplicationScope "Forest"

# Lister les redirecteurs conditionnels
Get-DnsServerZone | Where-Object {$_.ZoneType -eq "Forwarder"}

# Supprimer un redirecteur conditionnel
Remove-DnsServerZone -Name "partenaire.com" -Force
```

> [!example] Cas d'usage des redirecteurs conditionnels
> - **Entreprise multi-sites** : Chaque site gère son domaine DNS, redirecteurs pour les autres sites
> - **Partenaires B2B** : Résolution des noms du partenaire via leurs serveurs DNS
> - **Migration** : Redirection temporaire pendant la migration de domaines

### Différence entre redirecteurs et récursion

| Aspect | Redirecteur | Récursion |
|--------|-------------|-----------|
| **Méthode** | Délègue la résolution à un autre serveur | Résout directement en interrogeant les serveurs racine |
| **Charge** | Plus faible (le forwarder fait le travail) | Plus élevée (résolution complète) |
| **Cache** | Un seul cache centralisé | Chaque serveur construit son cache |
| **Internet** | Recommandé (via ISP ou serveurs publics) | Possible mais plus lent |

```powershell
# Désactiver la récursion (force l'utilisation des redirecteurs uniquement)
Set-DnsServerRecursion -Enable $false

# Réactiver la récursion
Set-DnsServerRecursion -Enable $true
```

> [!tip] Bonnes pratiques
> - Utilisez les serveurs DNS de votre FAI ou des services publics fiables (Google, Cloudflare, Quad9)
> - Configurez au moins 2 redirecteurs pour la redondance
> - Dans un environnement d'entreprise, centralisez les requêtes externes via des redirecteurs
> - Activez le "reordering" pour utiliser automatiquement le redirecteur le plus rapide

> [!warning] Sécurité
> - Les redirecteurs peuvent voir TOUTES vos requêtes DNS
> - Utilisez des services de confiance et considérez DNS sur TLS/HTTPS (DoT/DoH) si disponible
> - Ne redirigez jamais vers des serveurs DNS non fiables

---

## 🔧 Dépannage DNS

### Outils de diagnostic

#### nslookup

Outil interactif de requête DNS.

```bash
# Requête simple
nslookup www.entreprise.local

# Spécifier le serveur DNS à interroger
nslookup www.entreprise.local 192.168.1.10

# Recherche inversée
nslookup 192.168.1.20

# Mode interactif
nslookup
> server 192.168.1.10
> set type=MX
> entreprise.local
> set type=SRV
> _ldap._tcp.entreprise.local
> exit
```

#### Resolve-DnsName (PowerShell)

Alternative moderne à nslookup.

```powershell
# Résolution standard
Resolve-DnsName -Name "www.entreprise.local"

# Spécifier le type d'enregistrement
Resolve-DnsName -Name "entreprise.local" -Type MX

# Utiliser un serveur DNS spécifique
Resolve-DnsName -Name "www.entreprise.local" -Server "192.168.1.10"

# Désactiver la récursion (requête non récursive)
Resolve-DnsName -Name "www.entreprise.local" -NoRecursion

# Afficher les détails complets
Resolve-DnsName -Name "www.entreprise.local" -DnsOnly -Verbose
```

#### Test-DnsServer

Tester la disponibilité et les performances d'un serveur DNS.

```powershell
# Test complet du serveur DNS
Test-DnsServer -IPAddress "192.168.1.10"

# Test d'une zone spécifique
Test-DnsServer -IPAddress "192.168.1.10" -ZoneName "entreprise.local"

# Test avec rapport détaillé
Test-DnsServer -IPAddress "192.168.1.10" -Detailed
```

#### dnscmd

Outil en ligne de commande classique pour gérer DNS.

```bash
# Afficher les statistiques du serveur
dnscmd /statistics

# Vider le cache DNS
dnscmd /clearcache

# Lister les zones
dnscmd /enumzones

# Afficher les informations d'une zone
dnscmd /zoneinfo entreprise.local

# Recharger une zone depuis le fichier ou AD
dnscmd /zonerefresh entreprise.local
```

### Vider le cache DNS

```powershell
# Vider le cache du serveur DNS
Clear-DnsServerCache -Force

# Vider le cache du client (sur la machine locale)
ipconfig /flushdns

# Afficher le cache client
ipconfig /displaydns
```

> [!tip] Quand vider le cache ?
> - Après modification d'enregistrements DNS
> - Lors de problèmes de résolution persistants
> - Après une migration de serveur
> - En cas de suspicion de cache empoisonné

### Journaux DNS

```powershell
# Activer la journalisation des requêtes
Set-DnsServerDiagnostics -Queries $true -Answers $true

# Activer la journalisation complète
Set-DnsServerDiagnostics -All $true

# Consulter les paramètres de diagnostic
Get-DnsServerDiagnostics

# Localiser les journaux DNS
# Observateur d'événements : Applications and Services Logs > Microsoft > Windows > DNS-Server
```

**Événements importants à surveiller :**

| ID Événement | Description |
|--------------|-------------|
| **2** | Démarrage du service DNS |
| **4** | Arrêt du service DNS |
| **150** | Échec de chargement d'une zone |
| **407** | Enregistrement dynamique refusé |
| **6527** | Serveur en attente d'AD DS |

### Problèmes courants et solutions

#### ❌ Le serveur DNS ne répond pas

```powershell
# Vérifier que le service est démarré
Get-Service -Name DNS
Start-Service -Name DNS

# Vérifier les interfaces d'écoute
Get-DnsServer | Select-Object -ExpandProperty ServerSetting | Select-Object ListenAddresses

# Tester la connectivité réseau
Test-NetConnection -ComputerName "192.168.1.10" -Port 53
```

#### ❌ Les clients ne peuvent pas résoudre les noms

```powershell
# Sur le client, vérifier la configuration DNS
Get-DnsClientServerAddress

# Forcer le réenregistrement DNS du client
ipconfig /registerdns

# Vérifier que le serveur accepte les requêtes récursives
Get-DnsServerRecursion

# Tester la résolution depuis le serveur lui-même
Resolve-DnsName -Name "www.entreprise.local" -Server localhost
```

#### ❌ Les enregistrements SRV d'Active Directory sont manquants

```powershell
# Vérifier la présence des enregistrements SRV
Resolve-DnsName -Name "_ldap._tcp.dc._msdcs.entreprise.local" -Type SRV

# Forcer le réenregistrement (sur le DC)
ipconfig /registerdns

# Redémarrer le service Netlogon (recrée les enregistrements)
Restart-Service -Name Netlogon

# Vérifier les journaux du service Netlogon
Get-EventLog -LogName System -Source Netlogon -Newest 20
```

#### ❌ Les transferts de zone échouent

```powershell
# Vérifier les paramètres de transfert
Get-DnsServerZone -Name "entreprise.local" | Select-Object SecureSecondaries, SecondaryServers

# Vérifier les notifications
Get-DnsServerZone -Name "entreprise.local" | Select-Object Notify, NotifyServers

# Tester manuellement un transfert (sur le secondaire)
Start-DnsServerZoneTransfer -Name "entreprise.local" -FullTransfer
```

#### ❌ Résolution lente

```powershell
# Vérifier les redirecteurs et leur réactivité
Get-DnsServerForwarder
Test-NetConnection -ComputerName "8.8.8.8" -Port 53

# Analyser les statistiques du serveur
Get-DnsServerStatistics

# Augmenter la taille du cache si nécessaire
Set-DnsServerCache -MaxTTL (New-TimeSpan -Hours 24)

# Vérifier les indications de racine
Get-DnsServerRootHint
```

### Tests de validation complets

```powershell
# Script de validation DNS globale
function Test-DNSHealth {
    param($ZoneName = "entreprise.local")
    
    Write-Host "=== Test de santé DNS pour $ZoneName ===" -ForegroundColor Cyan
    
    # Test 1 : Service DNS
    Write-Host "
[Test 1] Service DNS..." -ForegroundColor Yellow
    Get-Service -Name DNS
    
    # Test 2 : Zones chargées
    Write-Host "
[Test 2] Zones chargées..." -ForegroundColor Yellow
    Get-DnsServerZone | Select-Object ZoneName, ZoneType, DynamicUpdate
    
    # Test 3 : Enregistrements critiques
    Write-Host "
[Test 3] Enregistrements SRV Active Directory..." -ForegroundColor Yellow
    Resolve-DnsName "_ldap._tcp.$ZoneName" -Type SRV
    
    # Test 4 : Redirecteurs
    Write-Host "
[Test 4] Redirecteurs configurés..." -ForegroundColor Yellow
    Get-DnsServerForwarder
    
    # Test 5 : Résolution externe
    Write-Host "
[Test 5] Résolution externe..." -ForegroundColor Yellow
    Resolve-DnsName "www.google.com"
    
    Write-Host "
=== Tests terminés ===" -ForegroundColor Cyan
}

# Exécuter les tests
Test-DNSHealth
```

> [!tip] Checklist de dépannage
> 1. ✅ Le service DNS est-il démarré ?
> 2. ✅ Les zones sont-elles chargées correctement ?
> 3. ✅ Les enregistrements critiques existent-ils ?
> 4. ✅ Le pare-feu autorise-t-il le port 53 (TCP et UDP) ?
> 5. ✅ Les clients utilisent-ils le bon serveur DNS ?
> 6. ✅ Les redirecteurs sont-ils accessibles ?
> 7. ✅ Le cache est-il corrompu ?

---

## 📚 Résumé des commandes PowerShell essentielles

```powershell
# === GESTION DES ZONES ===
Add-DnsServerPrimaryZone -Name "domaine.local" -ReplicationScope "Domain"
Get-DnsServerZone
Remove-DnsServerZone -Name "domaine.local"

# === ENREGISTREMENTS DNS ===
Add-DnsServerResourceRecordA -Name "serveur" -ZoneName "domaine.local" -IPv4Address "192.168.1.20"
Add-DnsServerResourceRecordCName -Name "www" -ZoneName "domaine.local" -HostNameAlias "serveur.domaine.local"
Add-DnsServerResourceRecordMX -Name "." -ZoneName "domaine.local" -MailExchange "mail.domaine.local" -Preference 10
Get-DnsServerResourceRecord -ZoneName "domaine.local"
Remove-DnsServerResourceRecord -ZoneName "domaine.local" -Name "serveur" -RRType A

# === REDIRECTEURS ===
Set-DnsServerForwarder -IPAddress "8.8.8.8","1.1.1.1"
Add-DnsServerConditionalForwarderZone -Name "autre.domaine" -MasterServers "10.0.0.1"

# === TRANSFERTS DE ZONE ===
Set-DnsServerPrimaryZone -Name "domaine.local" -SecureSecondaries "TransferToSecureServers" -SecondaryServers "192.168.1.11"

# === DIAGNOSTIC ===
Resolve-DnsName -Name "serveur.domaine.local"
Test-DnsServer -IPAddress "192.168.1.10"
Clear-DnsServerCache
Set-DnsServerDiagnostics -Queries $true

# === SCAVENGING ===
Set-DnsServerScavenging -ScavengingState $true -ScavengingInterval (New-TimeSpan -Days 7)
Set-DnsServerZoneAging -Name "domaine.local" -Aging $true
Start-DnsServerScavenging
```

---

> [!success] Points clés à retenir
> - Le DNS est **indispensable** au fonctionnement d'Active Directory
> - Les zones intégrées AD offrent réplication multimaître et sécurité renforcée
> - Les enregistrements SRV localisent les services AD (LDAP, Kerberos, GC)
> - Activez toujours les mises à jour dynamiques **sécurisées**
> - Configurez le scavenging pour nettoyer les enregistrements obsolètes
> - Utilisez des redirecteurs pour optimiser la résolution externe
> - Surveillez les journaux DNS et testez régulièrement la résolution
