# 🌐 Cours : DHCP & DNS — Niveau TSSR

> **Objectif** : Comprendre le fonctionnement de DHCP et DNS, savoir les configurer, les dépanner et identifier rapidement l'origine d'un problème.

---

## PARTIE 1 — DHCP

---

## 1. Qu'est-ce que le DHCP ?

**DHCP = Dynamic Host Configuration Protocol**

C'est le service qui **distribue automatiquement des adresses IP** aux machines qui rejoignent un réseau. Sans DHCP, il faudrait configurer manuellement l'IP de chaque équipement.

### Ce que le DHCP fournit à un client

| Information | Exemple | Rôle |
|-------------|---------|------|
| Adresse IP | 192.168.1.50 | Identifiant de la machine |
| Masque de sous-réseau | 255.255.255.0 | Délimite le réseau |
| Passerelle par défaut | 192.168.1.1 | Routeur pour sortir du réseau |
| Serveur DNS | 192.168.1.1 ou 8.8.8.8 | Résolution des noms |
| Durée du bail | 24h, 8j… | Durée de validité de l'IP |

---

## 2. Comment fonctionne le DHCP — Le processus DORA

Le DHCP utilise **4 étapes** pour attribuer une IP. On les retient avec l'acronyme **DORA**.

```
Client                          Serveur DHCP
  |                                  |
  |  1. DISCOVER (broadcast) ──────→ |   "Y a-t-il un serveur DHCP ici ?"
  |                                  |
  |  ←──── 2. OFFER ──────────────   |   "Oui, je te propose 192.168.1.50"
  |                                  |
  |  3. REQUEST (broadcast) ───────→ |   "J'accepte cette IP"
  |                                  |
  |  ←──── 4. ACK ────────────────   |   "C'est confirmé, bail de 24h"
  |                                  |
```

> 💡 Les étapes **DISCOVER** et **REQUEST** sont en **broadcast** car le client n'a pas encore d'IP. Il parle à tout le monde.

### Pourquoi le broadcast pose problème sur les grands réseaux ?

Le broadcast ne passe pas les routeurs. Si le serveur DHCP est sur un réseau différent du client, il faut configurer un **DHCP Relay Agent** (ou IP Helper) sur le routeur pour relayer les requêtes DHCP entre les réseaux.

---

## 3. Le bail DHCP — durée et renouvellement

Quand un client reçoit une IP, il reçoit aussi une **durée de bail (lease time)**. Une fois ce délai écoulé, l'IP peut être réattribuée à quelqu'un d'autre.

### Cycle de vie d'un bail

```
Attribution → 50% du bail → Tentative de renouvellement (unicast au serveur)
           → 87.5% du bail → Si pas de réponse, broadcast pour chercher un autre serveur
           → 100% → IP libérée, le client redemande via DORA
```

> 💡 En pratique, si le serveur DHCP est éteint pendant moins de 50% de la durée du bail, les clients ne s'en aperçoivent même pas.

---

## 4. Configuration DHCP — les éléments clés

### Pool d'adresses
La plage d'IP que le serveur peut distribuer.
Ex : `192.168.1.100` à `192.168.1.200`

### Exclusions
Les adresses dans le pool que le serveur **ne doit pas distribuer** — typiquement les imprimantes, serveurs, ou équipements réseau avec IP fixe.
Ex : exclure `192.168.1.100` à `192.168.1.110` pour les serveurs.

### Réservations (IP statique via DHCP)
On peut lier une adresse MAC à une IP fixe. Le client obtient toujours la même IP via DHCP, sans configuration manuelle.
Ex : Imprimante avec MAC `AA:BB:CC:DD:EE:FF` → toujours `192.168.1.101`

### Options DHCP courantes

| Option | Numéro | Description |
|--------|--------|-------------|
| Router (passerelle) | 3 | IP de la passerelle par défaut |
| DNS Server | 6 | Adresse du/des serveurs DNS |
| Domain Name | 15 | Nom de domaine (ex: entreprise.local) |
| Lease Time | 51 | Durée du bail en secondes |
| NTP Server | 42 | Serveur de temps |

---

## PARTIE 2 — DNS

---

## 5. Qu'est-ce que le DNS ?

**DNS = Domain Name System**

C'est le service qui **traduit les noms de domaine en adresses IP**. Sans DNS, il faudrait taper des IP à la place des noms de sites.

```
Utilisateur tape : www.google.com
DNS traduit en  : 142.250.74.68
Le navigateur se connecte à l'IP
```

> 💡 **Analogie** : Le DNS est l'annuaire téléphonique d'Internet. Tu cherches un nom, il te donne le numéro (l'IP).

---

## 6. La hiérarchie DNS

Le DNS est organisé en arbre, de droite à gauche dans un nom de domaine.

```
                        . (racine)
                       / \
                     .com  .fr  .org  ...
                    /
               google.com
              /
         www.google.com
```

Pour résoudre `www.google.com`, le résolveur interroge dans cet ordre :
1. Serveurs racine (`.`) → "qui gère `.com` ?"
2. Serveurs TLD `.com` → "qui gère `google.com` ?"
3. Serveurs de `google.com` → "quelle est l'IP de `www` ?"

---

## 7. Les types d'enregistrements DNS — à connaître absolument

| Type | Nom complet | Rôle | Exemple |
|------|------------|------|---------|
| **A** | Address | Nom → IPv4 | `www.site.com → 93.184.216.34` |
| **AAAA** | Address v6 | Nom → IPv6 | `www.site.com → 2606:2800::1` |
| **CNAME** | Canonical Name | Alias vers un autre nom | `ftp.site.com → www.site.com` |
| **MX** | Mail Exchanger | Serveur de messagerie | `site.com → mail.site.com` |
| **PTR** | Pointer | IPv4 → Nom (DNS inverse) | `93.184.216.34 → www.site.com` |
| **NS** | Name Server | Serveur DNS autoritaire de la zone | `site.com → ns1.site.com` |
| **SOA** | Start of Authority | Infos sur la zone DNS | Numéro de série, TTL… |
| **TXT** | Text | Texte libre | SPF, DKIM, vérification de domaine |

> ⚠️ **MX et TXT sont critiques pour la messagerie** : sans MX, pas d'email reçu. Sans SPF/DKIM dans les TXT, les emails partent en spam.

---

## 8. Résolution DNS — comment ça se passe réellement

Quand tu tapes `www.google.com` dans ton navigateur :

```
1. Le PC vérifie son cache DNS local        → réponse instantanée si déjà connu
2. Le PC vérifie le fichier hosts local     → /etc/hosts ou C:\Windows\System32\drivers\etc\hosts
3. Le PC interroge son serveur DNS (résolveur)
4. Le résolveur vérifie son propre cache    → réponse rapide si déjà connu
5. Si pas en cache → résolution récursive vers les serveurs racine, TLD, puis autoritaire
6. La réponse remonte jusqu'au PC avec l'IP
7. Le PC met en cache la réponse (durée = TTL)
```

### Le TTL (Time To Live)
Chaque enregistrement DNS a un TTL en secondes. C'est la durée pendant laquelle la réponse est mise en cache.
- TTL court (300s = 5 min) → changements rapides mais plus de charge sur les serveurs DNS
- TTL long (86400s = 24h) → moins de requêtes mais propagation lente lors de changements

---

## 9. DNS en entreprise — zones et types de serveurs

### Zones DNS

| Zone | Rôle |
|------|------|
| Zone directe | Résout nom → IP (le plus courant) |
| Zone inverse | Résout IP → nom (PTR), utilisé pour les logs, la sécurité |

### Types de serveurs DNS

| Type | Rôle |
|------|------|
| **Primaire** | Fait autorité sur la zone, contient les enregistrements en lecture/écriture |
| **Secondaire** | Copie de sauvegarde du primaire, lecture seule, mise à jour par transfert de zone |
| **Résolveur/Récursif** | Interroge les serveurs pour le compte des clients (ex : serveur DNS de ton FAI) |
| **Cache** | Stocke les réponses pour accélérer les requêtes futures |
| **Forwarder** | Redirige les requêtes non résolues localement vers un autre serveur DNS |

---

## 10. Lien DHCP ↔ DNS — le DNS dynamique (DDNS)

En entreprise, quand un PC obtient une IP via DHCP, il faut que le DNS soit mis à jour automatiquement pour que le nom du PC soit résolvable. C'est le **DNS dynamique (DDNS)**.

```
PC démarre → DHCP lui donne 192.168.1.55
           → DHCP (ou le PC) met à jour le DNS : pc-jean = 192.168.1.55
           → Les autres machines peuvent faire ping pc-jean
```

C'est fondamental dans les environnements **Active Directory** où les machines doivent se retrouver par leur nom.

---

## PARTIE 3 — Scénarios de dépannage

---

### 🔴 Scénario 1 — Un PC obtient une adresse 169.254.x.x

**Situation** :
- PC allumé le matin, IP affichée : `169.254.23.45`
- Impossible d'accéder au réseau ou à Internet
- La veille, tout fonctionnait

**Questions à se poser** :
1. Le câble réseau est-il branché ? La LED clignote-t-elle ?
2. Le serveur DHCP est-il démarré et joignable ?
3. Le PC est-il sur le bon VLAN (celui où se trouve le DHCP) ?
4. Y a-t-il un autre équipement qui joue au DHCP (rogue DHCP) ?

**Analyse** :

`169.254.x.x` = APIPA = le PC n'a reçu **aucune réponse** à son DISCOVER DHCP.

| Couche | État | Pourquoi |
|--------|------|---------|
| 1 | ? | À vérifier en premier |
| 2 | ? | VLAN correct ? |
| 3 (DHCP) | ❌ | Pas de réponse au DISCOVER |

**Marche à suivre** :
1. Vérifier physiquement le câble et la LED de la carte réseau
2. Tester avec une IP statique temporaire → si le réseau fonctionne, le problème est bien DHCP
3. Vérifier l'état du service DHCP sur le serveur
4. Vérifier les logs DHCP → y a-t-il des requêtes qui arrivent ?
5. Vérifier le VLAN et le DHCP Relay si le serveur est sur un autre réseau

**Commandes utiles** :
```bash
ipconfig /release             # Libérer l'IP actuelle
ipconfig /renew               # Forcer une nouvelle demande DHCP
ipconfig /all                 # Voir tous les détails réseau
```

> 💡 **Piège classique** : Un switch mal configuré qui ne laisse pas passer le broadcast DHCP. Toujours vérifier le VLAN.

---

### 🔴 Scénario 2 — Deux PC ont la même adresse IP

**Situation** :
- Des utilisateurs signalent des coupures réseau aléatoires
- En vérifiant, deux machines ont la même IP : `192.168.1.50`
- L'une est un PC fixe configuré en IP statique, l'autre est un PC portable en DHCP

**Questions à se poser** :
1. L'IP statique du PC fixe est-elle dans la plage du pool DHCP ?
2. Y a-t-il des exclusions configurées sur le serveur DHCP pour les IP statiques ?
3. Le serveur DHCP vérifie-t-il si l'IP est libre avant de l'attribuer (ping avant attribution) ?

**Analyse** :

Le serveur DHCP a attribué `192.168.1.50` au portable sans savoir qu'elle était déjà utilisée en statique.

| Cause | Explication |
|-------|------------|
| IP statique dans la plage DHCP | Le DHCP peut distribuer cette IP |
| Pas d'exclusion configurée | Le serveur ne sait pas qu'elle est réservée |

**Marche à suivre** :
1. **Solution immédiate** : changer l'IP statique du PC fixe (hors plage DHCP) OU forcer l'IP via une réservation DHCP
2. **Solution structurelle** : définir clairement deux zones :
   - IP statiques : `192.168.1.1` → `192.168.1.49`
   - Pool DHCP : `192.168.1.50` → `192.168.1.254`
   - OU exclure les IP statiques du pool DHCP
3. Préférer les **réservations DHCP** aux IP statiques manuelles — plus facile à gérer

> ⚠️ **Bonne pratique** : Ne jamais mettre d'IP statique manuelle dans la plage du pool DHCP. Toujours exclure ou utiliser des réservations.

---

### 🔴 Scénario 3 — Le serveur DHCP tombe en panne

**Situation** :
- Le serveur DHCP plante à 14h
- Les PC déjà connectés continuent de fonctionner
- Les nouveaux PC qui se connectent à partir de 14h n'obtiennent pas d'IP

**Questions à se poser** :
1. Quelle est la durée des baux existants ?
2. À quel moment les PC existants vont-ils tenter de renouveler ?
3. Y a-t-il un serveur DHCP de secours (failover) ?

**Analyse** :

Les PC déjà connectés ont un bail valide → ils continuent à fonctionner jusqu'à 50% de la durée du bail, moment où ils tenteront un renouvellement. Si le serveur est toujours en panne à ce moment-là, ils vont progressivement perdre leur IP.

| Durée du bail | Renouvellement tenté à | Impact si DHCP toujours KO à |
|--------------|----------------------|------------------------------|
| 8 heures | 14h + 4h = 18h | 14h + 7h = 21h, IP libérée |
| 24 heures | 14h + 12h = 2h du matin | 14h + 21h = 11h lendemain |

**Marche à suivre** :
1. Redémarrer le service DHCP ou le serveur en urgence
2. Si impossible → configurer temporairement un DHCP de secours sur un autre serveur
3. À long terme : mettre en place le **DHCP Failover** (deux serveurs DHCP synchronisés)
4. Augmenter la durée des baux pour gagner du temps en cas de panne

---

### 🔴 Scénario 4 — Ping IP OK, ping nom de domaine KO

**Situation** :
- `ping 8.8.8.8` → ✅ répond
- `ping google.com` → ❌ "Impossible de trouver l'hôte google.com"
- Tous les autres PC du réseau naviguent normalement

**Questions à se poser** :
1. Quel serveur DNS est configuré sur ce PC ?
2. Ce serveur DNS est-il joignable ?
3. Le serveur DNS répond-il aux requêtes ?
4. Y a-t-il une entrée incorrecte dans le cache DNS local ou le fichier hosts ?

**Analyse** :

| Couche | État | Pourquoi |
|--------|------|---------|
| 1 à 4 | ✅ | Ping IP externe OK |
| 7 (DNS) | ❌ | Résolution impossible |

**Marche à suivre** :
1. `ipconfig /all` → vérifier l'IP du serveur DNS configuré
2. `ping <ip-serveur-dns>` → le DNS est-il joignable ?
3. `nslookup google.com` → tester la résolution, voir le message d'erreur
4. `ipconfig /flushdns` → vider le cache DNS local (peut résoudre des entrées corrompues)
5. Si DNS mal configuré → corriger (souvent reçu via DHCP, vérifier les options DHCP)

**Commandes utiles** :
```bash
ipconfig /all                         # Voir le DNS configuré
ipconfig /flushdns                    # Vider le cache DNS
nslookup google.com                   # Tester la résolution
nslookup google.com 8.8.8.8           # Forcer le test sur 8.8.8.8
```

> 💡 Si `nslookup google.com 8.8.8.8` fonctionne mais `nslookup google.com` échoue → le serveur DNS configuré est le problème, pas Internet.

---

### 🔴 Scénario 5 — Un site interne inaccessible par son nom

**Situation** :
- Les utilisateurs accèdent à `http://intranet.entreprise.local`
- Ce matin, le site ne répond plus par son nom
- En testant avec l'IP directe `http://192.168.1.100` → ✅ ça fonctionne
- Internet fonctionne normalement

**Questions à se poser** :
1. L'enregistrement DNS `intranet.entreprise.local` existe-t-il toujours ?
2. L'IP dans l'enregistrement DNS est-elle correcte ?
3. Le serveur DNS interne fonctionne-t-il ?
4. Un changement récent a-t-il eu lieu (migration serveur, changement IP) ?

**Analyse** :

| Couche | État | Pourquoi |
|--------|------|---------|
| 1 à 4 | ✅ | Accès par IP OK |
| 7 (DNS interne) | ❌ | Enregistrement manquant ou IP incorrecte |

**Marche à suivre** :
1. `nslookup intranet.entreprise.local` → quelle IP retourne-t-il ?
2. Si IP incorrecte → corriger l'enregistrement A dans la zone DNS
3. Si aucune réponse → l'enregistrement a été supprimé → le recréer
4. Après correction → `ipconfig /flushdns` sur les postes clients pour vider le cache

**Commandes utiles** :
```bash
nslookup intranet.entreprise.local        # Vérifier ce que retourne le DNS
ipconfig /flushdns                        # Vider le cache DNS client
# Sur le serveur DNS Windows :
dnscmd /enumrecords entreprise.local @    # Lister tous les enregistrements de la zone
```

> 💡 **Cause classique** : Le serveur a été migré avec une nouvelle IP, mais l'enregistrement DNS n'a pas été mis à jour. L'accès par IP fonctionne, mais pas par nom.

---

### 🔴 Scénario 6 — Les emails envoyés arrivent en spam

**Situation** :
- Les utilisateurs envoient des emails depuis `@entreprise.com`
- Les destinataires externes les reçoivent dans leurs spams
- Les emails internes fonctionnent normalement
- Aucun changement récent signalé

**Questions à se poser** :
1. Y a-t-il un enregistrement SPF dans le DNS du domaine ?
2. DKIM est-il configuré et l'enregistrement TXT est-il présent ?
3. DMARC est-il configuré ?
4. Le serveur mail est-il dans une blacklist ?

**Analyse** :

Les filtres anti-spam des destinataires vérifient les enregistrements DNS pour s'assurer que l'email vient bien d'un serveur autorisé.

| Enregistrement | Rôle | Type DNS |
|---------------|------|---------|
| **SPF** | Liste les serveurs autorisés à envoyer pour ce domaine | TXT |
| **DKIM** | Signature cryptographique des emails | TXT |
| **DMARC** | Politique si SPF/DKIM échouent (rejeter, mettre en quarantaine…) | TXT |

**Marche à suivre** :
1. Vérifier le SPF : `nslookup -type=TXT entreprise.com` → chercher `v=spf1`
2. Vérifier DKIM : `nslookup -type=TXT default._domainkey.entreprise.com`
3. Tester sur `mxtoolbox.com` → outil complet pour vérifier SPF, DKIM, DMARC, blacklists
4. Si SPF absent → créer l'enregistrement TXT : `v=spf1 ip4:X.X.X.X include:_spf.google.com ~all`
5. Vérifier les blacklists → si listé, demander la délistage

**Commandes utiles** :
```bash
nslookup -type=TXT entreprise.com          # Voir les enregistrements TXT (SPF, DMARC)
nslookup -type=MX entreprise.com           # Voir le serveur de messagerie
nslookup -type=TXT default._domainkey.entreprise.com   # Vérifier DKIM
```

---

### 🔴 Scénario 7 — Lenteur de résolution DNS

**Situation** :
- Les pages web mettent du temps à s'afficher (3-5 secondes avant de charger)
- Une fois chargée, la page suivante du même site est rapide
- Le problème touche tous les utilisateurs du réseau
- Ping vers les IPs externes est rapide

**Questions à se poser** :
1. Le serveur DNS interne est-il surchargé ?
2. Le DNS interne utilise-t-il un forwarder lent ou injoignable ?
3. Y a-t-il un problème de cache DNS (TTL trop court) ?
4. Le DNS répond-il lentement sur le port 53 ?

**Analyse** :

La lenteur sur le premier accès et la rapidité sur les accès suivants est le signe classique d'un **problème de résolution DNS** (cache vide → résolution lente) vs **cache DNS plein** (réponse immédiate).

| Cause | Explication |
|-------|------------|
| Forwarder DNS lent ou KO | Le DNS interne attend une réponse d'un serveur externe lent |
| TTL très court | Le cache expire trop vite, résolution permanente |
| Serveur DNS surchargé | Temps de réponse élevé |

**Marche à suivre** :
1. Mesurer le temps de réponse DNS : `nslookup google.com` → noter le temps
2. Tester directement sur 8.8.8.8 : `nslookup google.com 8.8.8.8` → comparer
3. Si 8.8.8.8 est plus rapide → le forwarder du DNS interne est le problème
4. Changer le forwarder du serveur DNS interne pour pointer vers 8.8.8.8 ou 1.1.1.1
5. Vérifier la charge CPU/RAM du serveur DNS

**Commandes utiles** :
```bash
nslookup google.com                   # Mesurer le temps de résolution (DNS par défaut)
nslookup google.com 8.8.8.8           # Comparer avec Google DNS
Resolve-DnsName google.com            # PowerShell, plus de détails sur le temps
```

---

## 11. Exercices d'entraînement

---

**Exercice 1** — Un PC reçoit les informations DHCP suivantes :
- IP : `192.168.10.75`
- Masque : `255.255.255.0`
- Passerelle : `192.168.10.1`
- DNS : `192.168.10.5`
- Bail : 8 heures (obtenu à 9h00)

**Questions** :
1. À quelle heure le PC tentera-t-il de renouveler son bail ?
2. À quelle heure le bail expire-t-il définitivement si le renouvellement échoue ?
3. Si le DNS `192.168.10.5` tombe en panne, quel symptôme observera-t-on ?

<details>
<summary>👁️ Voir la réponse</summary>

1. Renouvellement à 50% du bail : 9h + 4h = **13h00**
2. Expiration à 100% : 9h + 8h = **17h00**
3. Le PC pourra toujours pinger par IP mais ne pourra plus résoudre les noms de domaine → sites web inaccessibles par leur nom, emails peut-être impactés.
</details>

---

**Exercice 2** — Un utilisateur dit "je ne peux pas accéder à `serveur-rh.entreprise.local` mais mon collègue à côté, lui, y accède sans problème."

Quelles sont les 3 premières vérifications à faire ?

<details>
<summary>👁️ Voir la réponse</summary>

1. `ipconfig /all` sur les deux PC → comparer les serveurs DNS configurés (peut-être que le PC en panne a un mauvais DNS)
2. `ipconfig /flushdns` sur le PC en panne → vider le cache, peut-être une entrée corrompue
3. `nslookup serveur-rh.entreprise.local` sur le PC en panne → voir quelle IP est retournée et si la résolution fonctionne
</details>

---

**Exercice 3** — Tu dois configurer le DHCP d'un nouveau réseau `10.0.5.0/24`. Les contraintes sont :
- Routeur : `10.0.5.1` (IP fixe)
- 3 serveurs : `10.0.5.2`, `10.0.5.3`, `10.0.5.4` (IP fixes)
- Imprimante réseau : toujours `10.0.5.20` (via DHCP)
- Le reste des machines : en DHCP dynamique

Définis la configuration DHCP optimale.

<details>
<summary>👁️ Voir la réponse</summary>

- **Pool DHCP** : `10.0.5.50` → `10.0.5.254`
- **Exclusions** : `10.0.5.1` → `10.0.5.49` (protège routeur, serveurs, et marge pour futurs équipements fixes)
- **Réservation** : MAC de l'imprimante → `10.0.5.20`
- **Options** : Passerelle = `10.0.5.1`, DNS = à définir selon l'infra
</details>

---

## 12. Aide-mémoire rapide

```
DHCP — PROCESSUS DORA
D → DISCOVER  : le client cherche un serveur (broadcast)
O → OFFER     : le serveur propose une IP
R → REQUEST   : le client accepte (broadcast)
A → ACK       : le serveur confirme le bail

DHCP — SYMPTÔMES CLASSIQUES
169.254.x.x          → Pas de réponse DHCP (APIPA)
IP en doublon        → IP statique dans la plage DHCP, pas d'exclusion
Pertes après panne   → Bail expiré, DHCP toujours KO à 50% du bail

DNS — ENREGISTREMENTS
A     → Nom → IPv4
AAAA  → Nom → IPv6
CNAME → Alias → Autre nom
MX    → Domaine → Serveur mail
PTR   → IP → Nom (DNS inverse)
TXT   → SPF, DKIM, DMARC

DNS — SYMPTÔMES CLASSIQUES
IP OK, nom KO                → DNS mal configuré ou en panne
Site interne inaccessible    → Enregistrement A manquant ou IP incorrecte
Emails en spam               → SPF/DKIM/DMARC absent ou incorrect
Lenteur premier chargement   → Forwarder DNS lent, cache vide

COMMANDES CLÉS
ipconfig /all          → Voir IP, masque, passerelle, DNS, bail DHCP
ipconfig /release      → Libérer l'IP DHCP
ipconfig /renew        → Redemander une IP au DHCP
ipconfig /flushdns     → Vider le cache DNS local
nslookup <nom>         → Tester la résolution DNS
nslookup <nom> <ip>    → Tester sur un DNS spécifique
```

---

> ✅ **À retenir** : DHCP et DNS sont les deux services silencieux qui font fonctionner tout le reste. Quand l'un d'eux plante, rien ne semble fonctionner — mais le réseau IP lui-même est souvent intact. La clé : toujours tester par IP d'abord pour isoler si c'est du DNS, puis vérifier le DHCP si pas d'IP du tout.
