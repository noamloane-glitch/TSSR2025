# DNS Linux (BIND) - Cheatsheet TSSR
> CCP 3 — Exploiter des serveurs Linux | CCP 4 — Réseau IP

## Concepts clés

| Concept | Définition |
|--------|-----------|
| BIND | Serveur DNS open source — paquet `bind9` (Debian/Ubuntu) |
| Zone directe | Nom → IP (`srv1.lan → 192.168.1.10`) |
| Zone inverse | IP → Nom — zone `in-addr.arpa` |
| Forwarder | Redirige les requêtes non résolues vers un DNS externe |
| Serveur autoritaire | Fait référence pour sa zone — répond depuis ses fichiers |
| `named` | Nom du processus/service BIND |
| Port | UDP/TCP **53** |

## Installation et service

| Action | Commande |
|--------|----------|
| Installer BIND9 | `apt install bind9 bind9utils` |
| Démarrer le service | `systemctl start named` |
| Activer au démarrage | `systemctl enable named` |
| Vérifier le statut | `systemctl status named` |
| Recharger la config (sans coupure) | `systemctl reload named` |
| Vérifier la config principale | `named-checkconf` |
| Vérifier un fichier de zone | `named-checkzone lab.lan /etc/bind/db.lab.lan` |

## Fichiers importants

| Fichier | Rôle |
|---------|------|
| `/etc/bind/named.conf` | Config principale — inclut les autres fichiers |
| `/etc/bind/named.conf.options` | Options globales (forwarders, recursion…) |
| `/etc/bind/named.conf.local` | Déclaration des zones personnalisées |
| `/etc/bind/db.lab.lan` | Fichier de zone directe (exemple) |
| `/etc/bind/db.192.168.1` | Fichier de zone inverse (exemple) |
| `/etc/resolv.conf` | Serveurs DNS utilisés par le système |
| `/etc/hosts` | Résolution statique locale — prioritaire sur DNS |
| `/etc/nsswitch.conf` | Ordre de résolution : `files` puis `dns` |
| `/var/log/syslog` | Journaux BIND (erreurs, requêtes) |

## Configuration type — named.conf.local

| Élément | Exemple |
|---------|---------|
| Déclarer zone directe | `zone "lab.lan" { type master; file "/etc/bind/db.lab.lan"; };` |
| Déclarer zone inverse | `zone "1.168.192.in-addr.arpa" { type master; file "/etc/bind/db.192.168.1"; };` |
| Forwarder | Dans `named.conf.options` : `forwarders { 8.8.8.8; 1.1.1.1; };` |
| Autoriser les requêtes | `allow-query { any; };` ou `allow-query { 192.168.1.0/24; };` |

## Enregistrements DNS (fichier de zone)

| Type | Exemple |
|------|---------|
| SOA | `@ IN SOA dns1.lab.lan. admin.lab.lan. ( 2024010101 3600 900 604800 86400 )` |
| NS | `@ IN NS dns1.lab.lan.` |
| A | `srv1 IN A 192.168.1.10` |
| CNAME | `www IN CNAME srv1.lab.lan.` |
| PTR (zone inverse) | `10 IN PTR srv1.lab.lan.` |
| MX | `@ IN MX 10 mail.lab.lan.` |

> ⚠ Les FQDN dans les fichiers de zone se terminent par un **point** `.` — l'oublier crée des erreurs silencieuses.

## Diagnostic

| Action | Commande |
|--------|----------|
| Résolution simple | `nslookup srv1.lab.lan` |
| Résolution avec serveur précis | `nslookup srv1.lab.lan 192.168.1.1` |
| Résolution complète | `dig srv1.lab.lan` |
| Résolution inverse | `dig -x 192.168.1.10` |
| Interroger un type précis | `dig MX lab.lan` |
| Tester la propagation pas à pas | `dig +trace domaine.com` |
| Vider le cache (systemd-resolved) | `systemctl restart systemd-resolved` |

## Points de vigilance

| Piège | Bonne pratique |
|-------|---------------|
| Point final manquant dans le fichier de zone | Tout FQDN doit se terminer par `.` |
| Serial SOA non incrémenté après modif | Toujours incrémenter — sinon les secondaires ne se synchronisent pas |
| Zone inverse absente | Créer la zone `in-addr.arpa` pour les enregistrements PTR |
| `recursion yes` en production publique | Risque d'amplification DNS — restreindre avec `allow-recursion` |
| `/etc/hosts` masque un problème DNS | Vérifier `nsswitch.conf` et vider le cache avant de diagnostiquer |
