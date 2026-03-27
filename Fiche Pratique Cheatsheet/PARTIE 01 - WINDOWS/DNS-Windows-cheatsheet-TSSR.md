# DNS Windows Server - Cheatsheet TSSR
> CCP 2 — Serveurs Windows & Active Directory | CCP 4 — Réseau IP

## Concepts clés

| Concept | Définition |
|--------|-----------|
| DNS Server | Rôle Windows Server — intégré à Active Directory |
| Zone intégrée AD | Fichier de zone stocké dans AD (réplication automatique entre DC) |
| Zone primaire | Fichier de zone local en lecture/écriture |
| Zone secondaire | Copie en lecture seule d'une zone primaire (transfert de zone) |
| Stub zone | Contient uniquement les NS d'une zone distante |
| Forwarder | Redirige les requêtes non résolues vers un DNS externe |
| Forwarder conditionnel | Redirige selon le domaine cible (ex. autre site) |
| `_msdcs` | Sous-domaine DNS critique pour la localisation des DC en AD |
| SRV records | Enregistrements créés automatiquement par AD (`_ldap`, `_kerberos`…) |
| Port | UDP/TCP **53** |

## Installation du rôle (PowerShell)

| Action | Commande |
|--------|----------|
| Installer le rôle DNS | `Install-WindowsFeature -Name DNS -IncludeManagementTools` |
| Installer avec ADDS | `Install-WindowsFeature -Name AD-Domain-Services,DNS -IncludeManagementTools` |
| Vérifier le service | `Get-Service DNS` |
| Démarrer / arrêter | `Start-Service DNS` / `Stop-Service DNS` |

## Gestion des zones (PowerShell)

| Action | Commande |
|--------|----------|
| Lister les zones | `Get-DnsServerZone` |
| Créer zone directe primaire | `Add-DnsServerPrimaryZone -Name "lab.lan" -ZoneFile "lab.lan.dns"` |
| Créer zone directe intégrée AD | `Add-DnsServerPrimaryZone -Name "lab.lan" -ReplicationScope "Domain"` |
| Créer zone inverse | `Add-DnsServerPrimaryZone -NetworkID "192.168.1.0/24" -ZoneFile "1.168.192.in-addr.arpa.dns"` |
| Supprimer une zone | `Remove-DnsServerZone -Name "lab.lan"` |
| Forcer la synchronisation AD | `Sync-DnsServerZone -Name "lab.lan"` |

## Gestion des enregistrements (PowerShell)

| Action | Commande |
|--------|----------|
| Ajouter un A | `Add-DnsServerResourceRecordA -ZoneName "lab.lan" -Name "srv1" -IPv4Address "192.168.1.10"` |
| Ajouter un PTR | `Add-DnsServerResourceRecordPtr -ZoneName "1.168.192.in-addr.arpa" -Name "10" -PtrDomainName "srv1.lab.lan"` |
| Ajouter un CNAME | `Add-DnsServerResourceRecordCName -ZoneName "lab.lan" -Name "www" -HostNameAlias "srv1.lab.lan"` |
| Lister les enregistrements | `Get-DnsServerResourceRecord -ZoneName "lab.lan"` |
| Supprimer un enregistrement | `Remove-DnsServerResourceRecord -ZoneName "lab.lan" -Name "srv1" -RRType "A"` |

## Fichiers et emplacements

| Élément | Emplacement |
|---------|-------------|
| Fichiers de zone (zone fichier) | `C:\Windows\System32\dns\` |
| Zones intégrées AD | Stockées dans AD (`CN=MicrosoftDNS` dans la partition annuaire) |
| Journaux DNS | Observateur d'événements → `DNS Server` |
| Cache DNS serveur | `Show-DnsServerCache` / `Clear-DnsServerCache` |
| Hosts local | `C:\Windows\System32\drivers\etc\hosts` |

## Diagnostic

| Action | Commande |
|--------|----------|
| Résolution simple | `Resolve-DnsName srv1.lab.lan` |
| Résolution avec serveur précis | `Resolve-DnsName srv1.lab.lan -Server 192.168.1.1` |
| Résolution inverse | `Resolve-DnsName 192.168.1.10` |
| Interroger un type précis | `Resolve-DnsName lab.lan -Type MX` |
| Vider le cache DNS client | `ipconfig /flushdns` |
| Afficher le cache DNS client | `ipconfig /displaydns` |
| Vider le cache DNS serveur | `Clear-DnsServerCache` |
| Tester avec nslookup | `nslookup srv1.lab.lan 192.168.1.1` |
| Vérifier les SRV AD | `nslookup -type=SRV _ldap._tcp.lab.lan` |

## Points de vigilance

| Piège | Bonne pratique |
|-------|---------------|
| Zone non intégrée AD sur un DC | Préférer les zones intégrées AD — réplication et sécurité automatiques |
| Enregistrements SRV AD absents | Vérifier avec `dcdiag /test:DNS` — les DC les inscrivent automatiquement |
| Forwarder non configuré | Sans forwarder, pas de résolution Internet depuis le réseau interne |
| DNS client pointant sur lui-même (DC) | Le DC doit pointer sur lui-même en DNS primaire, un autre DC en secondaire |
| Nettoyage dynamique (scavenging) désactivé | Active les enregistrements obsolètes — configurer le scavenging en prod |
| Zone inverse absente | Les PTR sont nécessaires pour certains services (messagerie, Kerberos) |
| `ipconfig /flushdns` inefficace sur le problème | Penser à vider aussi le cache **serveur** avec `Clear-DnsServerCache` |
