# 🌐 Cours : Modèle OSI & Dépannage par couche

> **Objectif** : Savoir identifier rapidement quelle couche OSI est en cause lors d'un problème réseau, et comprendre pourquoi.

---

## 1. Rappel express du modèle OSI

|#|Couche|Nom|Protocoles / Exemples|
|---|---|---|---|
|7|Application|Ce que voit l'utilisateur|HTTP, HTTPS, SSH, DNS, FTP, SMTP|
|6|Présentation|Encodage, chiffrement|SSL/TLS, JPEG, ASCII|
|5|Session|Gestion des sessions|NetBIOS, RPC|
|4|Transport|Communication bout en bout|TCP, UDP — **Ports**|
|3|Réseau|Routage entre réseaux|IP, ICMP (ping), ARP|
|2|Liaison|Communication sur le réseau local|Ethernet, MAC, Switch|
|1|Physique|Le signal brut|Câbles, Wi-Fi, fibres, hubs|

> 💡 **Moyen mémo-technique (de bas en haut)** : **P**hysique **L**iaison **R**éseau **T**ransport **S**ession **P**résentation **A**pplication → **"Pour Les Réseaux, Tout Semble Plus Accessible"**

---

## 2. La règle d'or du dépannage OSI

> **Toujours raisonner couche par couche, de bas en haut.**

Avant de soupçonner une couche haute, assure-toi que la couche en dessous fonctionne.

```
Couche 1 OK ? → Couche 2 OK ? → Couche 3 OK ? → Couche 4 OK ? → Couche 7 OK ?
```

Si tu sautes des étapes, tu risques de perdre du temps à chercher un problème de certificat SSL alors que le câble est juste débranché.

---

## 3. Les indices et ce qu'ils révèlent

|Indice observé|Ce que ça confirme|
|---|---|
|Le câble est branché, la LED clignote|Couche **1** ✅|
|Le PC obtient une adresse MAC, est visible sur le switch|Couche **2** ✅|
|Le PC reçoit une IP (DHCP)|Couche **2 et 3** ✅|
|`ping` vers la passerelle fonctionne|Couche **3** locale ✅|
|`ping` vers une IP externe fonctionne|Couche **3** complète ✅|
|Connexion TCP établie sur un port (ex : port 22)|Couche **4** ✅|
|Service SSH/HTTP accessible et répond correctement|Couche **7** ✅|

---

## 4. Protocoles : où vivent-ils vraiment ?

C'est souvent là que la confusion arrive. Un protocole vit en couche 7, mais il utilise un port en couche 4.

|Protocole|Couche applicative|Port (couche 4)|
|---|---|---|
|HTTP|7|80|
|HTTPS|7|443|
|SSH|7|22|
|DNS|7|53|
|FTP|7|21|
|SMTP|7|25|

### ⚠️ La distinction clé

- **Le port ne répond pas** (connexion refusée ou timeout) → problème **couche 4** : service non démarré, port fermé, firewall.
- **Le port répond mais le service est cassé** (erreur HTTP 500, certificat invalide…) → problème **couche 7**.

**Analogie 📞** :

- Port fermé = le téléphone ne sonne même pas.
- Service cassé = quelqu'un décroche mais raccroche aussitôt.

---

## 5. Scénarios de dépannage commentés

---

### 🔴 Scénario 1 — Ping passerelle OK, ping 8.8.8.8 KO

**Situation** :

- IP reçue automatiquement ✅
- Ping passerelle ✅
- Ping 8.8.8.8 ❌

**Analyse couche par couche** :

|Couche|État|Pourquoi|
|---|---|---|
|1-2|✅|IP reçue = DHCP a fonctionné|
|3 (local)|✅|Ping passerelle OK|
|3 (externe)|❌|Impossible d'atteindre 8.8.8.8|

**Couche suspectée** : **Couche 3** (routage vers l'extérieur)

**Causes probables** :

- Pas de route par défaut configurée sur le routeur
- Problème NAT (la translation d'adresse ne se fait pas)
- Lien WAN down (panne côté FAI)
- Firewall bloquant le trafic sortant

---

### 🔴 Scénario 2 — Accès par IP OK, accès par nom de domaine KO

**Situation** :

- `http://192.168.1.10` fonctionne ✅
- `http://monsite.com` ne fonctionne pas ❌

**Analyse couche par couche** :

|Couche|État|Pourquoi|
|---|---|---|
|1 à 4|✅|La connexion IP fonctionne|
|7 (DNS)|❌|La résolution du nom échoue|

**Couche suspectée** : **Couche 7 — service DNS**

**Causes probables** :

- Serveur DNS en panne
- Mauvais serveur DNS configuré sur le poste
- DHCP fournit une adresse DNS incorrecte

> 💡 Pour tester : `nslookup monsite.com` ou `dig monsite.com` — si ça échoue, c'est DNS.

---

### 🔴 Scénario 3 — Ping OK, port 22 OK, port 443 KO

**Situation** :

- Ping serveur ✅
- SSH (port 22) ✅
- HTTPS (port 443) ❌

**Analyse couche par couche** :

|Couche|État|Pourquoi|
|---|---|---|
|1-2-3|✅|Ping OK|
|4 (port 22)|✅|SSH accessible|
|4 (port 443)|❌|Port 443 ne répond pas|

**Couche suspectée** : **Couche 4 d'abord**, puis couche 7 si le port répond.

**Causes probables** :

- Service HTTPS non démarré (`nginx`, `apache` stoppé)
- Port 443 bloqué par un firewall
- Certificat SSL invalide/expiré → dans ce cas le port répond mais la couche 7 échoue

> 💡 Pour tester : `telnet IP 443` ou `curl -v https://IP` — si la connexion TCP ne s'établit pas → couche 4. Si elle s'établit mais plante → couche 7.

---

### 🔴 Scénario 4 — Rien ne fonctionne, pas d'IP

**Situation** :

- Pas d'adresse IP ❌
- Rien ne ping ❌

**Analyse** : On n'atteint même pas la couche 3.

**Couche suspectée** : **Couche 1 ou 2**

**Causes probables** :

- Câble débranché ou défectueux (couche 1)
- Mauvais port de switch, VLAN incorrect (couche 2)
- Service DHCP en panne (couche 3, mais l'absence d'IP peut aussi venir de là)

---

### 🔴 Scénario 5 — Intermittence réseau, pertes de paquets

**Situation** :

- Ping fonctionne mais avec des pertes (ex : 30% de paquets perdus)
- Connexions qui se coupent aléatoirement

**Couche suspectée** : **Couche 1 ou 2**

**Causes probables** :

- Câble endommagé (couche 1)
- Problème de duplex/vitesse sur le port switch (couche 2)
- Interférences Wi-Fi (couche 1)
- Carte réseau défaillante (couche 1-2)

---

## 6. Scénarios par couche OSI — Entraînement complet

> Pour chaque scénario : lis la situation, pose-toi les questions, puis lis l'analyse.

---

### 🟤 COUCHE 1 — Physique

---

#### Scénario C1-A — Le PC ne s'allume pas sur le réseau

**Situation** :

- Un technicien branche un nouveau PC sur une prise murale
- Aucune LED ne s'allume sur la carte réseau
- Le PC n'obtient pas d'IP
- Les autres PC du même bureau fonctionnent normalement

**Questions à se poser** :

1. Est-ce que la LED de la carte réseau clignote ?
2. Le câble est-il bien branché des deux côtés (PC et prise murale) ?
3. Le câble a-t-il été testé avec un autre PC ?
4. La prise murale est-elle reliée au switch (câble en baie de brassage) ?

**Analyse** :

|Couche|État|Pourquoi|
|---|---|---|
|1|❌|Pas de signal physique détecté|
|2, 3, 4, 7|❌|Impossible sans couche 1|

**Couche suspectée** : **Couche 1**

**Marche à suivre** :

1. Changer le câble RJ45 → si LED s'allume, c'était le câble
2. Tester la prise murale avec un câble qui fonctionne ailleurs
3. Vérifier en baie de brassage que le port est bien raccordé au switch
4. Si rien ne fonctionne → carte réseau défaillante

**Commandes utiles** :

```bash
ip link show          # Vérifier si l'interface est UP ou DOWN
ethtool eth0          # Vérifier la vitesse et l'état du lien
```

> 💡 **Indice clé couche 1** : Si la LED ne clignote pas, ne cherche pas plus loin. C'est physique.

---

#### Scénario C1-B — Wi-Fi instable, déconnexions fréquentes

**Situation** :

- Un utilisateur se connecte en Wi-Fi
- La connexion fonctionne mais se coupe toutes les 10-15 minutes
- En se rapprochant du point d'accès, le problème disparaît
- Les autres utilisateurs proches n'ont pas ce problème

**Questions à se poser** :

1. Quelle est la force du signal Wi-Fi sur le poste ?
2. Y a-t-il des obstacles (murs épais, équipements métalliques) entre le PC et la borne ?
3. Est-ce que d'autres appareils Bluetooth ou micro-ondes sont à proximité ? (interférences sur 2.4GHz)
4. Le problème disparaît si on branche un câble Ethernet ?

**Analyse** :

|Couche|État|Pourquoi|
|---|---|---|
|1|❌ (instable)|Signal Wi-Fi faible ou interféré|
|2, 3+|Instable|Conséquence directe|

**Couche suspectée** : **Couche 1 (signal radio)**

**Marche à suivre** :

1. Vérifier la puissance du signal (RSSI) : en dessous de -70 dBm c'est faible
2. Changer le canal Wi-Fi pour éviter les interférences
3. Rapprocher l'utilisateur du point d'accès ou installer un répéteur
4. Tester avec un câble pour confirmer que c'est bien du Wi-Fi

---

### 🟠 COUCHE 2 — Liaison de données

---

#### Scénario C2-A — Deux PC sur le même switch ne se voient pas

**Situation** :

- PC-A et PC-B sont branchés sur le même switch
- Les deux ont une adresse IP dans le même réseau (192.168.1.x)
- Les LEDs des ports switch sont allumées pour les deux
- PC-A ne peut pas pinger PC-B

**Questions à se poser** :

1. Les deux PC sont-ils dans le même VLAN ?
2. La table ARP de PC-A contient-elle l'adresse MAC de PC-B ?
3. Un pare-feu Windows bloque-t-il les pings sur l'un des PC ?
4. Les adresses IP sont-elles bien dans le même sous-réseau (même masque) ?

**Analyse** :

|Couche|État|Pourquoi|
|---|---|---|
|1|✅|LEDs allumées|
|2|❌|Les trames ne passent pas entre les ports|
|3|❌|Conséquence|

**Couche suspectée** : **Couche 2 (VLAN ou configuration switch)**

**Marche à suivre** :

1. Vérifier que les deux ports sont dans le même VLAN sur le switch
2. Faire `arp -a` sur PC-A : est-ce que l'IP de PC-B est résolue en MAC ?
3. Si pas de MAC → le switch ne fait pas passer les trames → problème VLAN
4. Si MAC présente mais ping KO → vérifier le pare-feu du PC-B

**Commandes utiles** :

```bash
arp -a                        # Table ARP locale
ping -c 4 192.168.1.X         # Test basique
```

---

#### Scénario C2-B — Le réseau est lent uniquement pour un PC

**Situation** :

- Tout le réseau fonctionne normalement
- Un seul PC a des débits très faibles (1-2 Mbps au lieu de 100 Mbps)
- Ping local avec des temps élevés (50-100ms au lieu de <1ms)
- Pas de perte de connectivité complète

**Questions à se poser** :

1. Y a-t-il des erreurs sur le port switch correspondant ?
2. Le port switch et la carte réseau sont-ils négociés à la même vitesse ?
3. Le câble a-t-il été testé ?
4. Y a-t-il un conflit d'adresse MAC sur le réseau ?

**Analyse** :

|Couche|État|Pourquoi|
|---|---|---|
|1|✅ (à moitié)|Connexion établie mais signal dégradé|
|2|❌|Négociation duplex incorrecte ou erreurs de trame|
|3+|Lent|Conséquence|

**Couche suspectée** : **Couche 2 (duplex mismatch)**

**Marche à suivre** :

1. Vérifier avec `ethtool eth0` la vitesse négociée (doit être 100Mbps Full Duplex)
2. Forcer la vitesse/duplex côté PC ET côté switch (ou laisser auto-négociation des deux côtés)
3. Changer le câble RJ45

> 💡 **Duplex mismatch** : Si un côté est en Full Duplex et l'autre en Half Duplex, les performances chutent drastiquement mais la connexion ne se coupe pas complètement.

---

### 🟡 COUCHE 3 — Réseau

---

#### Scénario C3-A — Un PC ne peut joindre qu'une partie du réseau

**Situation** :

- PC dans le réseau 192.168.1.0/24
- Peut pinger tous les PC de son réseau ✅
- Ne peut pas pinger les PC du réseau 192.168.2.0/24 ❌
- Les PC du réseau 192.168.2.0/24 se voient entre eux

**Questions à se poser** :

1. Le PC a-t-il une passerelle (gateway) configurée ?
2. La passerelle est-elle joignable ?
3. Le routeur a-t-il des routes vers 192.168.2.0/24 ?
4. Existe-t-il une règle de firewall bloquant le trafic inter-réseau ?

**Analyse** :

|Couche|État|Pourquoi|
|---|---|---|
|1-2|✅|Communication locale OK|
|3 (local)|✅|Ping dans le même réseau OK|
|3 (inter-réseau)|❌|Pas de route ou firewall|

**Couche suspectée** : **Couche 3 (routage inter-réseau)**

**Marche à suivre** :

1. `ip route show` → vérifier qu'une route par défaut existe
2. Pinger la passerelle : `ping 192.168.1.254`
3. `traceroute 192.168.2.10` → voir où les paquets s'arrêtent
4. Sur le routeur, vérifier les routes statiques et les règles de firewall

**Commandes utiles** :

```bash
ip route show                    # Table de routage du PC
traceroute 192.168.2.10          # Tracer le chemin
ping 192.168.1.254               # Tester la passerelle
```

---

#### Scénario C3-B — Adresse IP en conflit

**Situation** :

- Un utilisateur allume son PC
- Une notification Windows indique "Conflit d'adresse IP détecté"
- Le réseau fonctionne de façon aléatoire, parfois OK parfois non
- D'autres utilisateurs se plaignent aussi de problèmes intermittents

**Questions à se poser** :

1. L'IP du PC est-elle configurée en statique ou en DHCP ?
2. Y a-t-il un autre équipement sur le réseau avec la même IP ?
3. Le serveur DHCP a-t-il attribué deux fois la même IP ?
4. Y a-t-il des équipements avec IP statique dans la plage DHCP ?

**Analyse** :

|Couche|État|Pourquoi|
|---|---|---|
|1-2|✅|Signal et trame OK|
|3|❌|Deux équipements ont la même IP → le routage est confus|

**Couche suspectée** : **Couche 3 (conflit IP)**

**Marche à suivre** :

1. `arp -a` → chercher deux entrées avec la même IP et deux MAC différentes
2. Identifier l'équipement en conflit (scanner réseau : `arp-scan`)
3. Si IP statique sur le PC → la changer ou passer en DHCP
4. Sur le serveur DHCP → vérifier les exclusions et les baux

---

### 🟢 COUCHE 4 — Transport

---

#### Scénario C4-A — Application accessible en interne, pas depuis l'extérieur

**Situation** :

- Un serveur web tourne sur le port 8080 en interne
- Accessible depuis le LAN : `http://192.168.1.50:8080` ✅
- Inaccessible depuis Internet : `http://monip-publique:8080` ❌
- Ping vers l'IP publique fonctionne

**Questions à se poser** :

1. Y a-t-il une règle de redirection de port (port forwarding) sur le routeur ?
2. Le port 8080 est-il ouvert dans le firewall du routeur ?
3. Le pare-feu du serveur lui-même accepte-t-il les connexions sur ce port ?
4. L'ISP bloque-t-il ce port ?

**Analyse** :

|Couche|État|Pourquoi|
|---|---|---|
|1-2-3|✅|Ping IP publique OK|
|4|❌|Le port 8080 n'est pas accessible de l'extérieur|
|7|Non testable|On n'atteint pas la couche 4|

**Couche suspectée** : **Couche 4 (port forwarding / firewall)**

**Marche à suivre** :

1. Sur le routeur → ajouter une règle NAT/PAT : port 8080 externe → 192.168.1.50:8080
2. Vérifier le firewall du routeur (autoriser le port 8080 entrant)
3. Vérifier le firewall du serveur : `iptables -L` ou pare-feu Windows
4. Tester depuis l'extérieur : `telnet monip-publique 8080`

**Commandes utiles** :

```bash
telnet <IP> 8080              # Tester si le port répond
ss -tuln | grep 8080          # Vérifier que le service écoute bien sur ce port
```

---

#### Scénario C4-B — Connexions TCP qui tombent aléatoirement

**Situation** :

- Des sessions SSH se coupent après quelques minutes d'inactivité
- Les téléchargements s'interrompent parfois au milieu
- Le ping fonctionne en permanence
- Le problème touche plusieurs utilisateurs

**Questions à se poser** :

1. Y a-t-il un timeout configuré sur un firewall ou un équipement réseau ?
2. Les sessions TCP keepalive sont-elles activées ?
3. Y a-t-il de la perte de paquets sur le réseau ?
4. Un équipement réseau (NAT, firewall) ferme-t-il les connexions inactives ?

**Analyse** :

|Couche|État|Pourquoi|
|---|---|---|
|1-2-3|✅|Ping permanent OK|
|4|❌ (intermittent)|Les sessions TCP sont coupées|

**Couche suspectée** : **Couche 4 (timeout TCP / keepalive)**

**Marche à suivre** :

1. Activer le TCP keepalive sur le client SSH (`ServerAliveInterval 60` dans `~/.ssh/config`)
2. Sur le firewall → augmenter le timeout des sessions TCP (souvent 30-60 min par défaut)
3. Vérifier s'il y a de la perte de paquets : `ping -c 1000 passerelle | tail`
4. Vérifier les logs du firewall pour voir s'il coupe explicitement les connexions

---

### 🔵 COUCHE 5 — Session

---

#### Scénario C5-A — L'utilisateur est déconnecté de son application métier

**Situation** :

- Un utilisateur travaille sur une application client-serveur (ERP, base de données)
- Après 30 minutes sans action, il est déconnecté
- En reprenant le travail, il doit se reconnecter et perd ses données non sauvegardées
- Le réseau fonctionne parfaitement (ping, navigation web OK)

**Questions à se poser** :

1. L'application a-t-elle un timeout de session configuré ?
2. Y a-t-il un intermédiaire (proxy, load balancer) qui coupe les connexions longues ?
3. Le serveur d'application gère-t-il les sessions persistantes ?
4. Le client est-il derrière un NAT qui expire les entrées ?

**Analyse** :

|Couche|État|Pourquoi|
|---|---|---|
|1 à 4|✅|Réseau fonctionnel|
|5|❌|La session applicative est expirée ou coupée|
|7|❌ (conséquence)|L'application perd le contexte|

**Couche suspectée** : **Couche 5 (gestion de session)**

**Marche à suivre** :

1. Augmenter le timeout de session côté serveur d'application
2. Configurer un keepalive applicatif (heartbeat) pour maintenir la session ouverte
3. Vérifier les paramètres du NAT ou du proxy qui pourrait couper les connexions longues
4. Mettre en place une sauvegarde automatique côté application

---

### 🟣 COUCHE 6 — Présentation

---

#### Scénario C6-A — Fichiers reçus illisibles ou corrompus

**Situation** :

- Un utilisateur envoie un fichier Word à un collègue par email
- Le collègue reçoit le fichier mais il s'ouvre avec des caractères illisibles (symboles, ?)
- D'autres fichiers (PDF, images) passent sans problème
- Les deux utilisateurs ont des versions différentes de Word

**Questions à se poser** :

1. Quel encodage est utilisé pour le fichier ? (UTF-8, Latin-1, Windows-1252 ?)
2. Les versions de logiciels sont-elles compatibles ?
3. Le fichier est-il corrompu pendant le transfert ou avant ?
4. Un antivirus ou un proxy a-t-il modifié le fichier ?

**Analyse** :

|Couche|État|Pourquoi|
|---|---|---|
|1 à 5|✅|Le fichier arrive à destination|
|6|❌|Le format ou l'encodage n'est pas interprété correctement|
|7|❌ (conséquence)|L'application ne peut pas afficher le contenu|

**Couche suspectée** : **Couche 6 (encodage / format)**

**Marche à suivre** :

1. Vérifier si le problème est reproductible avec un autre format (`.txt`, `.pdf`)
2. Enregistrer le fichier Word en mode de compatibilité ou en PDF avant envoi
3. Vérifier l'encodage du fichier texte si c'est un `.txt` ou `.csv`
4. Mettre à jour les logiciels pour assurer la compatibilité des formats

---

#### Scénario C6-B — Erreur de certificat SSL sur un site web

**Situation** :

- Un utilisateur accède à `https://intranet.entreprise.com`
- Le navigateur affiche "Votre connexion n'est pas privée" / erreur certificat
- Le site était accessible hier
- Les autres sites HTTPS fonctionnent normalement

**Questions à se poser** :

1. Le certificat SSL a-t-il expiré ?
2. Le nom de domaine correspond-il au certificat (Common Name) ?
3. La date/heure du poste client est-elle correcte ?
4. Le certificat est-il signé par une autorité reconnue (ou auto-signé) ?

**Analyse** :

|Couche|État|Pourquoi|
|---|---|---|
|1 à 4|✅|La connexion TCP s'établit|
|6|❌|La couche TLS/SSL échoue lors du handshake|
|7|❌ (conséquence)|HTTPS ne peut pas s'établir|

**Couche suspectée** : **Couche 6 (SSL/TLS)**

**Marche à suivre** :

1. Vérifier la date d'expiration du certificat : `openssl s_client -connect intranet.entreprise.com:443`
2. Renouveler le certificat s'il est expiré (Let's Encrypt, PKI interne)
3. Vérifier que la date/heure du client est correcte (un décalage horaire peut invalider un certificat)
4. Si auto-signé → importer le certificat dans le magasin de certificats du client

**Commandes utiles** :

```bash
openssl s_client -connect monsite.com:443   # Inspecter le certificat SSL
curl -v https://monsite.com                  # Voir le détail de la négociation TLS
```

---

### 🔴 COUCHE 7 — Application

---

#### Scénario C7-A — L'application web affiche une erreur 500

**Situation** :

- Un utilisateur accède à une application web
- La page s'affiche mais avec une erreur "500 Internal Server Error"
- D'autres pages du même site fonctionnent
- Le problème est apparu après une mise à jour de l'application

**Questions à se poser** :

1. Les logs de l'application ou du serveur web indiquent-ils une erreur précise ?
2. La mise à jour a-t-elle introduit un bug ou une mauvaise configuration ?
3. La base de données est-elle accessible depuis le serveur web ?
4. Les permissions des fichiers ont-elles changé ?

**Analyse** :

|Couche|État|Pourquoi|
|---|---|---|
|1 à 4|✅|La connexion TCP est établie, le serveur répond|
|6|✅|HTTPS fonctionne (la page s'affiche partiellement)|
|7|❌|L'application plante côté serveur|

**Couche suspectée** : **Couche 7 (application)**

**Marche à suivre** :

1. Consulter les logs applicatifs : `/var/log/nginx/error.log`, logs de l'app
2. Vérifier si la base de données répond : `mysql -u user -p -h db-server`
3. Tester un rollback de la mise à jour
4. Vérifier les permissions des fichiers de l'application

---

#### Scénario C7-B — Emails envoyés mais non reçus

**Situation** :

- Les utilisateurs envoient des emails sans erreur côté client
- Les destinataires externes ne reçoivent rien (ni en boîte de réception, ni en spam)
- Les emails internes (même domaine) fonctionnent
- La messagerie externe fonctionnait hier

**Questions à se poser** :

1. Le serveur de messagerie est-il listé dans une blacklist anti-spam ?
2. Les enregistrements DNS (MX, SPF, DKIM, DMARC) sont-ils corrects ?
3. Le port 25 (SMTP) est-il ouvert en sortie depuis le serveur mail ?
4. Y a-t-il des erreurs dans les logs du serveur SMTP ?

**Analyse** :

|Couche|État|Pourquoi|
|---|---|---|
|1 à 4|✅|Réseau OK, emails internes OK|
|7 (SMTP sortant)|❌|Les emails sont bloqués ou rejetés en externe|

**Couche suspectée** : **Couche 7 (SMTP / configuration DNS mail)**

**Marche à suivre** :

1. Vérifier les blacklists : `mxtoolbox.com/blacklists.aspx`
2. Tester les enregistrements DNS : `nslookup -type=MX mondomaine.com`
3. Vérifier SPF : `nslookup -type=TXT mondomaine.com`
4. Consulter les logs SMTP pour voir les erreurs de rejet

**Commandes utiles** :

```bash
nslookup -type=MX mondomaine.com        # Vérifier les enregistrements MX
telnet smtp.mondomaine.com 25            # Tester la connexion SMTP
```

---

## 7. Méthode de diagnostic — Le réflexe à avoir

Face à n'importe quel problème réseau, pose-toi ces questions **dans l'ordre** :

```
1. 🔌 Le câble est branché ? La LED clignote ?          → Couche 1
2. 🖧  Le switch voit la machine ? L'IP est reçue ?      → Couche 2
3. 🌐  La passerelle répond au ping ?                    → Couche 3 locale
4. 🌍  Une IP externe répond au ping ?                   → Couche 3 externe
5. 🚪  Le port du service est-il ouvert ?                → Couche 4
6. ⚙️  Le service répond correctement ?                  → Couche 7
```

Dès qu'une étape échoue → **tu as trouvé ta couche**, inutile d'aller plus loin.

---

## 8. Commandes utiles par couche

|Couche|Commande|Ce qu'elle teste|
|---|---|---|
|1-2|`ip link` / `ifconfig`|État de l'interface réseau|
|2|`arp -a`|Table ARP (correspondance IP ↔ MAC)|
|3|`ping <IP>`|Connectivité IP|
|3|`traceroute <IP>`|Chemin emprunté par les paquets|
|3|`ip route`|Table de routage|
|4|`telnet <IP> <port>`|Est-ce que le port répond ?|
|4|`netstat -tuln`|Liste des ports ouverts en local|
|4|`ss -tuln`|Idem (plus moderne)|
|7|`nslookup <domaine>`|Résolution DNS|
|7|`curl -v <URL>`|Requête HTTP complète avec détails|

---

## 9. Résumé visuel — tous les scénarios

```
SYMPTÔME OBSERVÉ                              COUCHE   EXEMPLES DE CAUSES
──────────────────────────────────────────────────────────────────────────────
LED éteinte, pas de signal                →   1        Câble KO, carte réseau HS
Wi-Fi instable, coupures fréquentes       →   1        Signal faible, interférences
PC invisible sur le réseau local          →   2        VLAN incorrect, port switch KO
Réseau lent pour 1 seul PC               →   2        Duplex mismatch, câble dégradé
Ping passerelle OK, ping externe KO       →   3        Pas de route, NAT KO, FAI
Conflit IP, comportement aléatoire        →   3        Deux équipements avec même IP
Un seul réseau inaccessible               →   3        Route manquante sur le routeur
Port inaccessible depuis l'extérieur      →   4        Port forwarding manquant
Sessions qui tombent aléatoirement        →   4        Timeout TCP, firewall agressif
Sessions expirées, déconnexion app        →   5        Timeout session, keepalive absent
Fichiers reçus corrompus/illisibles       →   6        Encodage, incompatibilité format
Erreur certificat SSL                     →   6        Certificat expiré, TLS KO
IP OK mais nom de domaine KO              →   7        DNS en panne, mauvaise config
Erreur 500 sur une app web                →   7        Bug applicatif, BDD inaccessible
Emails envoyés non reçus en externe       →   7        Blacklist, SPF/DKIM incorrect
```

---

> ✅ **À retenir** : Le modèle OSI n'est pas juste un truc à mémoriser pour l'exam. C'est un outil de dépannage. Chaque indice que tu as (ping, port, DNS…) te permet d'éliminer des couches et de cibler le problème rapidement.
> 
> 🔑 **La règle qui ne change jamais** : dès qu'un symptôme apparaît, commence par le bas (couche 1) et remonte. Ne saute jamais d'étape.