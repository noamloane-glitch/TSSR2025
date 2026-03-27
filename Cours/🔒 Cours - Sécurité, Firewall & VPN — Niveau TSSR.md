# 🔒 Cours : Sécurité, Firewall & VPN — Niveau TSSR

> **Objectif** : Comprendre les principes de la sécurité réseau, maîtriser le fonctionnement des firewalls, des VPN, et savoir identifier et répondre aux menaces courantes.

---

## PARTIE 1 — Les bases de la sécurité réseau

---

## 1. Les trois piliers de la sécurité — CIA

Toute réflexion en sécurité informatique repose sur trois principes fondamentaux, regroupés sous le sigle **CIA** :

|Pilier|Nom français|Définition|Exemple de menace|
|---|---|---|---|
|**C** — Confidentiality|Confidentialité|Seules les personnes autorisées accèdent aux données|Interception de données (sniffing)|
|**I** — Integrity|Intégrité|Les données ne sont pas modifiées sans autorisation|Modification d'un fichier en transit (MITM)|
|**A** — Availability|Disponibilité|Les services sont accessibles quand on en a besoin|Attaque DDoS|

> 💡 Chaque mesure de sécurité que tu mets en place protège un ou plusieurs de ces trois piliers. Un firewall protège la **disponibilité** et la **confidentialité**. Le chiffrement protège la **confidentialité** et l'**intégrité**.

---

## 2. Les types d'attaques — à connaître pour l'exam

### Attaques réseau

|Attaque|Description|Contre-mesure|
|---|---|---|
|**DoS / DDoS**|Saturer un service de requêtes pour le rendre indisponible|Filtrage, rate limiting, anti-DDoS|
|**Man in the Middle (MITM)**|Se placer entre deux hôtes pour intercepter/modifier le trafic|Chiffrement (TLS), certificats|
|**ARP Spoofing**|Empoisonner la table ARP pour détourner le trafic|Dynamic ARP Inspection sur les switchs|
|**Sniffing**|Capturer et lire les paquets qui transitent sur le réseau|Chiffrement, segmentation réseau|
|**Port Scanning**|Scanner les ports ouverts d'une cible pour trouver des vulnérabilités|Firewall, IDS/IPS|

### Attaques applicatives

|Attaque|Description|Contre-mesure|
|---|---|---|
|**Phishing**|Email frauduleux pour voler des credentials|Sensibilisation, filtrage email, MFA|
|**Ransomware**|Chiffre les fichiers et demande une rançon|Sauvegardes, EDR, mises à jour|
|**SQL Injection**|Injecter du code SQL dans un formulaire web|WAF, validation des entrées|
|**Brute Force**|Tester toutes les combinaisons de mots de passe|Verrouillage de compte, MFA|
|**Pass the Hash**|Utiliser le hash du mot de passe sans le connaître|Kerberos, protection LSASS|

### Attaques internes

|Attaque|Description|Contre-mesure|
|---|---|---|
|**Élévation de privilèges**|Obtenir des droits supérieurs à ceux attribués|Principe du moindre privilège|
|**Insider threat**|Employé malveillant qui exfiltre des données|Journalisation, DLP, séparation des rôles|

---

## 3. Les principes de sécurité fondamentaux

|Principe|Description|
|---|---|
|**Moindre privilège**|Donner uniquement les droits nécessaires, pas plus|
|**Défense en profondeur**|Multiplier les couches de sécurité (firewall + antivirus + EDR + MFA…)|
|**Segmentation réseau**|Isoler les réseaux sensibles (VLAN, DMZ) pour limiter la propagation|
|**Zero Trust**|Ne jamais faire confiance par défaut, toujours vérifier l'identité|
|**Journalisation**|Tracer toutes les actions pour détecter et investiguer les incidents|
|**Mise à jour régulière**|Corriger les vulnérabilités connues (patch management)|

---

## PARTIE 2 — Le Firewall

---

## 4. Qu'est-ce qu'un firewall ?

Un **firewall (pare-feu)** est un équipement ou logiciel qui **filtre le trafic réseau** selon des règles définies par l'administrateur. Il décide ce qui peut entrer et sortir du réseau.

### Les types de firewall

|Type|Description|Niveau OSI|
|---|---|---|
|**Stateless (sans état)**|Analyse chaque paquet individuellement selon src/dst IP et port|Couche 3-4|
|**Stateful (avec état)**|Suit l'état des connexions TCP — sait si un paquet fait partie d'une session légitime|Couche 3-4|
|**Applicatif (WAF / NGFW)**|Analyse le contenu applicatif — détecte les injections SQL, les malwares...|Couche 7|
|**Proxy firewall**|Interrompt la connexion et la recrée — agit comme intermédiaire|Couche 7|

> 💡 **Stateful vs Stateless** : Un firewall stateless voit un paquet de retour (ACK) et ne sait pas s'il correspond à une connexion légitime. Un firewall stateful le sait, car il suit l'état de la connexion. Le stateful est bien plus sécurisé.

---

## 5. Les règles de firewall — comment ça fonctionne

Une règle de firewall est composée de :

```
[Source IP] [Destination IP] [Port] [Protocole] [Action : ALLOW / DENY]
```

### Ordre des règles — critique

Les règles sont évaluées **de haut en bas**. La **première règle qui correspond** est appliquée, les suivantes sont ignorées.

```
Règle 1 : Autoriser 192.168.1.10 → n'importe où → port 443 → ALLOW
Règle 2 : Refuser tout → port 443 → DENY
Règle 3 : Autoriser tout → n'importe où → ALLOW    ← jamais atteinte pour le port 443
```

### La règle implicite finale

Tout firewall bien configuré a une **règle "deny all" implicite** à la fin : tout ce qui n'est pas explicitement autorisé est refusé.

```
... tes règles d'autorisation ...
Règle finale (implicite) : Tout → Tout → DENY  ←  toujours présente
```

### Sens du filtrage

|Sens|Description|
|---|---|
|**Inbound (entrant)**|Trafic qui arrive depuis l'extérieur vers le réseau interne|
|**Outbound (sortant)**|Trafic qui part du réseau interne vers l'extérieur|

> ⚠️ Beaucoup d'administrateurs filtrent uniquement l'entrant et oublient l'outbound. Or un malware qui s'exfiltre vers l'extérieur est du trafic **sortant**.

---

## 6. La DMZ — zone démilitarisée

La **DMZ (DeMilitarized Zone)** est une zone réseau intermédiaire entre Internet et le réseau interne. On y place les serveurs **accessibles depuis Internet** (web, mail, DNS public) tout en les **isolant du réseau interne**.

```
Internet ←→ [Firewall externe] ←→ DMZ ←→ [Firewall interne] ←→ LAN interne
                                    │
                              Serveur Web
                              Serveur Mail
                              Serveur DNS public
```

### Pourquoi c'est important ?

Si un attaquant compromet le serveur web en DMZ, il se retrouve dans la DMZ — pas directement dans le LAN interne. Le firewall interne le bloque.

> 💡 **Règle de base** : Un serveur en DMZ ne devrait jamais pouvoir initier de connexion vers le LAN interne. C'est toujours le LAN interne qui initie (ex : pour les mises à jour).

---

## 7. Les zones de sécurité

Les firewalls modernes définissent des **zones de sécurité** avec des niveaux de confiance différents :

|Zone|Niveau de confiance|Contenu|
|---|---|---|
|**WAN / Untrust**|0 — Pas de confiance|Internet|
|**DMZ**|50 — Confiance partielle|Serveurs publics|
|**LAN / Trust**|100 — Confiance totale|Réseau interne|
|**Management**|100+ — Très haute confiance|Administration des équipements|

---

## PARTIE 3 — Le VPN

---

## 8. Qu'est-ce qu'un VPN ?

Un **VPN (Virtual Private Network)** crée un **tunnel chiffré** entre deux points à travers un réseau non sécurisé (Internet). Il garantit la **confidentialité** et l'**intégrité** des données en transit.

### Les deux grands types de VPN

|Type|Description|Usage|
|---|---|---|
|**Site à site**|Relie deux réseaux d'entreprise entre eux de façon permanente|Deux bureaux distants qui partagent le même réseau|
|**Client à site**|Un utilisateur individuel se connecte au réseau de l'entreprise|Télétravail|

---

## 9. Les protocoles VPN — comparatif

|Protocole|Port|Chiffrement|Avantages|Inconvénients|
|---|---|---|---|---|
|**IPsec**|UDP 500, 4500|AES, 3DES|Standard, robuste, supporte site à site|Configuration complexe|
|**SSL/TLS (OpenVPN)**|TCP/UDP 443|AES|Passe les firewalls facilement (port 443)|Plus lent que IPsec|
|**L2TP/IPsec**|UDP 1701 + 500|IPsec|Compatible nativement sur Windows|Double encapsulation = lent|
|**WireGuard**|UDP 51820|ChaCha20|Très rapide, simple, moderne|Plus récent, moins répandu en entreprise|
|**PPTP**|TCP 1723|MPPE (faible)|Simple, natif Windows|❌ Obsolète, non sécurisé — à éviter|

> ⚠️ **PPTP est compromis** — ne jamais l'utiliser en production. Préférer IPsec ou OpenVPN.

---

## 10. IPsec — fonctionnement détaillé

IPsec sécurise les communications en deux phases :

### Phase 1 — Établissement du tunnel de contrôle (IKE)

- Authentification des deux équipements (clé pré-partagée ou certificat)
- Négociation des algorithmes de chiffrement
- Création d'un canal sécurisé pour la phase 2

### Phase 2 — Établissement du tunnel de données

- Négociation des paramètres pour chiffrer les données
- Création des SA (Security Associations)
- Début du transfert des données chiffrées

### Les deux modes IPsec

|Mode|Description|Usage|
|---|---|---|
|**Transport**|Chiffre uniquement le payload (données), pas l'en-tête IP|Communication entre deux hôtes|
|**Tunnel**|Chiffre tout le paquet IP et ajoute un nouvel en-tête|VPN site à site (le plus courant)|

### Les deux protocoles IPsec

|Protocole|Rôle|
|---|---|
|**AH (Authentication Header)**|Authentification et intégrité — mais **pas de chiffrement**|
|**ESP (Encapsulating Security Payload)**|Authentification, intégrité ET chiffrement — le plus utilisé|

---

## 11. VPN SSL/TLS — le plus utilisé pour le télétravail

Le VPN SSL utilise **TLS** (le même protocole que HTTPS) pour chiffrer le tunnel. C'est le type le plus utilisé pour les connexions client à site car :

- Passe à travers presque tous les firewalls (port 443)
- Ne nécessite souvent qu'un navigateur ou un client léger
- Facile à déployer pour les utilisateurs nomades

### Deux variantes

|Variante|Description|
|---|---|
|**SSL VPN Full Tunnel**|Tout le trafic de l'utilisateur passe par le VPN|
|**SSL VPN Split Tunnel**|Seul le trafic vers l'entreprise passe par le VPN, le reste va directement sur Internet|

> 💡 **Split tunnel** = moins de charge sur le VPN et meilleure expérience utilisateur. Mais attention : le trafic non-VPN n'est pas filtré par le firewall de l'entreprise.

---

## PARTIE 4 — Concepts complémentaires

---

## 12. IDS et IPS

|Terme|Nom complet|Rôle|
|---|---|---|
|**IDS**|Intrusion Detection System|Détecte les attaques et **alerte** — n'agit pas|
|**IPS**|Intrusion Prevention System|Détecte les attaques et **bloque** automatiquement|

Les IDS/IPS analysent le trafic à la recherche de **signatures d'attaques connues** ou de **comportements anormaux**.

### Positionnement dans le réseau

```
Internet → [Firewall] → [IPS] → LAN interne
                          ↑
              Analyse et bloque en temps réel
```

---

## 13. Les protocoles d'authentification

|Protocole|Description|Sécurité|
|---|---|---|
|**RADIUS**|Centralise l'authentification réseau (Wi-Fi, VPN, switches)|✅ Bon|
|**LDAP**|Interroge un annuaire (AD) pour authentifier|✅ Bon (LDAPS = chiffré)|
|**TACACS+**|Comme RADIUS mais sépare authentification/autorisation/comptabilité|✅ Très bon|
|**MFA**|Authentification multi-facteurs (mot de passe + OTP, carte, biométrie)|✅✅ Excellent|

> 💡 **MFA** (Multi-Factor Authentication) est aujourd'hui **indispensable** pour les accès VPN et les comptes privilégiés. Un mot de passe seul ne suffit plus.

---

## 14. Le chiffrement — bases essentielles

|Concept|Description|Exemple|
|---|---|---|
|**Symétrique**|Une seule clé pour chiffrer ET déchiffrer|AES — rapide, utilisé pour les données|
|**Asymétrique**|Paire de clés : publique (chiffre) + privée (déchiffre)|RSA — pour l'échange de clés, les certificats|
|**Hybride**|Asymétrique pour échanger une clé symétrique, symétrique pour les données|TLS (HTTPS)|
|**Hachage**|Fonction à sens unique — produit une empreinte unique|SHA-256, MD5 (obsolète)|

### Comment TLS fonctionne (simplifié)

```
1. Client → Serveur : "Bonjour, je supporte TLS 1.3, voici mes algorithmes"
2. Serveur → Client : "OK, voici mon certificat (clé publique)"
3. Client vérifie le certificat (signé par une CA de confiance ?)
4. Échange de clé de session (asymétrique)
5. Communication chiffrée avec la clé de session (symétrique AES)
```

---

## PARTIE 5 — Scénarios de dépannage

---

### 🔴 Scénario 1 — Le VPN s'établit mais aucun trafic ne passe

**Situation** :

- Un utilisateur en télétravail se connecte au VPN SSL de l'entreprise
- La connexion VPN s'établit correctement (client indique "Connecté")
- Il ne peut ni pinger ni accéder aux serveurs internes
- Sans VPN, Internet fonctionne normalement

**Questions à se poser** :

1. L'utilisateur reçoit-il bien une IP du pool VPN ?
2. Les routes vers le réseau interne sont-elles bien injectées par le VPN ?
3. Le firewall interne autorise-t-il le trafic venant du pool VPN ?
4. Le serveur VPN a-t-il une route de retour vers le pool VPN ?

**Analyse** :

|Vérification|Résultat possible|
|---|---|
|IP VPN reçue|Si pas d'IP → problème d'authentification ou de pool|
|Routes injectées|`route print` ou `ip route` — les réseaux internes apparaissent-ils ?|
|Firewall interne|Le pool VPN (ex: 10.8.0.0/24) est-il autorisé à atteindre le LAN ?|

**Marche à suivre** :

1. Vérifier l'IP obtenue : `ipconfig /all` → chercher l'interface VPN
2. `route print` (Windows) ou `ip route` (Linux) → les routes internes sont-elles là ?
3. Sur le firewall : vérifier qu'une règle autorise `pool_VPN → LAN interne`
4. Sur le serveur VPN : vérifier la configuration NAT ou les routes de retour
5. Tester avec `ping` depuis le client VPN vers l'IP d'un serveur interne

**Commandes utiles** :

```bash
ipconfig /all                     # Voir l'interface VPN et l'IP reçue
route print                       # Voir les routes injectées par le VPN
ping 192.168.1.10                 # Tester la connectivité vers un serveur interne
tracert 192.168.1.10              # Voir par où passe le trafic
```

> 💡 VPN connecté mais rien ne passe = problème de **routage** ou de **règle firewall**, pas de tunnel. Le tunnel est OK, c'est ce qu'il y a dedans qui bloque.

---

### 🔴 Scénario 2 — Le VPN IPsec site à site est en échec

**Situation** :

- Deux sites distants (Paris et Lyon) reliés par un VPN IPsec
- Le lien était fonctionnel hier
- Ce matin, les machines de Paris ne peuvent plus atteindre celles de Lyon
- Internet fonctionne sur les deux sites

**Questions à se poser** :

1. Le tunnel IPsec phase 1 (IKE) s'est-il établi ?
2. Le tunnel IPsec phase 2 s'est-il établi ?
3. Y a-t-il eu un changement sur le firewall/routeur d'un des sites ?
4. La clé pré-partagée (PSK) ou les paramètres IKE ont-ils changé ?
5. L'IP publique d'un des sites a-t-elle changé ?

**Analyse** :

|Phase|Si elle échoue|Cause probable|
|---|---|---|
|Phase 1 (IKE)|Pas de tunnel|PSK incorrecte, algorithmes incompatibles, IP publique changée|
|Phase 2|Tunnel phase 1 OK mais pas de données|Sélecteurs de trafic incorrects, algorithmes de chiffrement incompatibles|

**Marche à suivre** :

1. Vérifier les logs IPsec sur le firewall des deux côtés
2. Vérifier que les IP publiques n'ont pas changé (FAI avec IP dynamique ?)
3. Vérifier la correspondance des paramètres : PSK, algorithmes, durée de vie des SA
4. Effacer les SA existantes et forcer la renegociation
5. Vérifier les sélecteurs de trafic (les plages réseau déclarées des deux côtés)

> 💡 **Cause classique** : Une des IP publiques a changé (FAI avec DHCP) et la configuration pointe encore vers l'ancienne IP. Solution : utiliser des DNS dynamiques (DDNS) ou des IP publiques fixes.

---

### 🔴 Scénario 3 — Des règles firewall bloquent un service

**Situation** :

- Une nouvelle application a été déployée sur le serveur `192.168.1.100`, port `8443`
- Les utilisateurs du LAN ne peuvent pas y accéder
- Le serveur répond bien en local (testé directement sur le serveur)
- Un `telnet 192.168.1.100 8443` depuis un PC client échoue

**Questions à se poser** :

1. Y a-t-il une règle firewall qui autorise le port 8443 depuis le LAN vers le serveur ?
2. Le service écoute-t-il sur toutes les interfaces (`0.0.0.0`) ou seulement sur `127.0.0.1` ?
3. Le pare-feu Windows du serveur bloque-t-il ce port ?
4. Y a-t-il un autre équipement réseau (switch L3, routeur) qui filtre ce trafic ?

**Analyse** :

|Couche|État|Pourquoi|
|---|---|---|
|1-2-3|✅|Ping vers le serveur OK|
|4|❌|Telnet sur port 8443 échoue → port bloqué ou non ouvert|
|7|Non testable|On n'atteint pas le service|

**Marche à suivre** :

1. Sur le serveur : `ss -tuln | grep 8443` → le service écoute-t-il sur `0.0.0.0:8443` ?
2. Sur le serveur : vérifier le pare-feu Windows → le port 8443 est-il ouvert en entrée ?
3. Sur le firewall réseau : vérifier les règles → `LAN → Serveur → port 8443 → ALLOW` existe-t-elle ?
4. Ajouter la règle manquante et retester

**Commandes utiles** :

```bash
# Sur le serveur Linux
ss -tuln | grep 8443                    # Le port écoute-t-il ?
iptables -L -n | grep 8443              # Règle firewall Linux

# Sur le serveur Windows
netstat -ano | findstr 8443             # Le port écoute-t-il ?
netsh advfirewall show allprofiles      # État du pare-feu Windows

# Depuis le client
telnet 192.168.1.100 8443               # Test de connexion TCP
```

---

### 🔴 Scénario 4 — Suspicion d'une attaque MITM sur le réseau

**Situation** :

- Des utilisateurs signalent des erreurs de certificat sur des sites HTTPS internes
- Les connexions semblent plus lentes que d'habitude
- L'un des utilisateurs remarque que la passerelle ARP de son PC a changé

**Questions à se poser** :

1. La table ARP des PC est-elle cohérente ? (une IP = une MAC unique)
2. Y a-t-il un équipement inconnu sur le réseau qui répond aux requêtes ARP ?
3. Un switch présente-t-il du trafic anormal sur ses ports ?
4. Le Dynamic ARP Inspection est-il activé sur les switchs ?

**Analyse** :

Changement de passerelle ARP + erreurs de certificat = indicateurs classiques d'une attaque **ARP Spoofing** / **ARP Poisoning**.

```
Situation normale :
PC → ARP "Qui est 192.168.1.1 ?" → Routeur répond avec sa MAC

Attaque ARP Spoofing :
PC → ARP "Qui est 192.168.1.1 ?" → Attaquant répond avec SA propre MAC
→ Le trafic du PC passe par l'attaquant avant d'aller au routeur
```

**Marche à suivre** :

1. `arp -a` sur plusieurs PC → comparer les MAC associées à la passerelle — sont-elles identiques ?
2. Scanner le réseau pour trouver l'équipement inconnu (`arp-scan`, `nmap`)
3. Sur les switchs : activer **Dynamic ARP Inspection (DAI)** et **DHCP Snooping**
4. Isoler et identifier l'équipement attaquant (port du switch)
5. À long terme : segmenter le réseau par VLAN pour limiter la portée de ce type d'attaque

**Commandes utiles** :

```bash
arp -a                                  # Voir la table ARP du PC
arp-scan --localnet                     # Scanner tous les équipements du réseau local
nmap -sn 192.168.1.0/24                # Découverte réseau sans scan de ports
```

---

### 🔴 Scénario 5 — Utilisateurs victimes de phishing

**Situation** :

- Plusieurs utilisateurs ont reçu un email semblant venir du DSI
- L'email demandait de cliquer sur un lien et de saisir leurs credentials
- Deux utilisateurs ont cliqué et saisi leurs mots de passe
- Le domaine dans le lien était `connexion-entreprise.net` au lieu de `entreprise.com`

**Questions à se poser** :

1. Les comptes compromis sont-ils toujours actifs ?
2. Y a-t-il eu des connexions suspectes depuis ces comptes ?
3. L'email a-t-il contourné le filtre anti-spam ?
4. D'autres utilisateurs ont-ils cliqué ?

**Marche à suivre — réponse à incident** :

```
IMMÉDIAT (dans l'heure)
1. Désactiver les comptes compromis dans AD
2. Réinitialiser les mots de passe depuis un poste sain
3. Activer/vérifier le MFA sur les comptes compromis
4. Analyser les logs de connexion → y a-t-il eu des connexions suspectes ?

COURT TERME (dans la journée)
5. Identifier tous les destinataires de l'email
6. Vérifier si d'autres ont cliqué ou saisi des credentials
7. Bloquer le domaine frauduleux dans le filtre email et DNS
8. Analyser les postes des utilisateurs ayant cliqué (malware ?)

LONG TERME (prévention)
9. Renforcer le filtre anti-spam (DMARC, anti-phishing)
10. Sensibiliser les utilisateurs
11. Simuler des campagnes de phishing pour tester et former
```

> 💡 **Règle d'or** : En cas de suspicion de compromission, **désactiver d'abord, analyser ensuite**. Mieux vaut un utilisateur bloqué temporairement qu'un attaquant qui exfiltre des données.

---

### 🔴 Scénario 6 — Détection d'un scan de ports

**Situation** :

- L'IDS remonte des alertes : une IP externe `185.x.x.x` scanne de nombreux ports sur ton serveur web
- Les scans se font de façon progressive (pas brutale) pour éviter les alertes
- Le firewall n'a encore rien bloqué automatiquement

**Questions à se poser** :

1. L'IP source est-elle connue comme malveillante ? (blacklists)
2. Des ports sensibles sont-ils exposés inutilement ?
3. Le firewall doit-il bloquer automatiquement cette IP ?
4. Y a-t-il eu une tentative d'exploitation après le scan ?

**Marche à suivre** :

1. Bloquer immédiatement l'IP `185.x.x.x` sur le firewall
2. Vérifier les ports exposés sur Internet → fermer tout ce qui n'est pas nécessaire
3. Consulter les blacklists : `mxtoolbox.com`, `abuseipdb.com`
4. Analyser les logs du serveur web → y a-t-il eu des requêtes suspectes après le scan ?
5. Mettre en place le **port knocking** ou changer le port SSH (22 → port non standard)
6. Activer le blocage automatique des IP trop actives (fail2ban, IPS)

> 💡 **Un scan de ports n'est pas une attaque en soi** — c'est une phase de reconnaissance. Mais il indique qu'une attaque est peut-être en préparation. Agir vite.

---

## 15. Exercices d'entraînement

---

**Exercice 1** — Tu dois configurer un firewall pour un serveur web en DMZ. Le serveur doit :

- Être accessible depuis Internet sur les ports 80 et 443
- Pouvoir contacter le serveur de base de données en interne sur le port 3306
- Ne rien accepter d'autre depuis Internet
- Le LAN interne doit pouvoir l'administrer en SSH (port 22)

Décris les règles firewall à créer (entre Internet↔DMZ et DMZ↔LAN).

<details> <summary>👁️ Voir la réponse</summary>

**Règles Internet → DMZ** :

```
ALLOW  Any           → Serveur_Web  → TCP 80   (HTTP)
ALLOW  Any           → Serveur_Web  → TCP 443  (HTTPS)
DENY   Any           → DMZ         → Any       (tout le reste)
```

**Règles DMZ → LAN** :

```
ALLOW  Serveur_Web   → Serveur_BDD → TCP 3306  (MySQL)
DENY   DMZ           → LAN         → Any       (le serveur web ne peut pas initier d'autres connexions vers le LAN)
```

**Règles LAN → DMZ** :

```
ALLOW  LAN_Admin     → Serveur_Web → TCP 22    (SSH administration)
DENY   LAN           → DMZ         → Any       (le reste est bloqué)
```

</details>

---

**Exercice 2** — Quelle différence entre IDS et IPS ? Dans quel cas préfères-tu l'un ou l'autre ?

<details> <summary>👁️ Voir la réponse</summary>

- **IDS** : détecte et alerte — ne bloque pas. Idéal en mode "observation" pour analyser le trafic sans risque de faux positifs qui bloqueraient du trafic légitime.
- **IPS** : détecte et bloque automatiquement. Idéal en production quand les signatures sont fiables, car il coupe l'attaque en temps réel.

En pratique : on commence souvent en mode IDS pour affiner les règles et éviter les faux positifs, puis on passe en IPS une fois les règles validées.

</details>

---

**Exercice 3** — Un utilisateur dit que son VPN "se connecte mais qu'il ne peut pas accéder aux serveurs". Quelles sont tes 4 premières vérifications dans l'ordre ?

<details> <summary>👁️ Voir la réponse</summary>

1. `ipconfig /all` → L'interface VPN a-t-elle bien reçu une IP du pool VPN ?
2. `route print` → Les routes vers le réseau interne sont-elles injectées ?
3. `ping <IP-serveur-interne>` → Le ping passe-t-il par le tunnel ?
4. Vérifier sur le firewall → le pool VPN est-il autorisé à atteindre les serveurs internes ?

</details>

---

## 16. Aide-mémoire rapide

```
PILIERS DE LA SÉCURITÉ (CIA)
C → Confidentialité  : seuls les autorisés accèdent aux données
I → Intégrité        : les données ne sont pas altérées
A → Disponibilité    : les services sont accessibles

FIREWALL — PRINCIPES CLÉS
- Règles évaluées de haut en bas, première correspondance appliquée
- Deny All implicite en fin de liste
- Filtrer entrant ET sortant
- DMZ = zone intermédiaire pour les serveurs publics

VPN — PROTOCOLES
IPsec    → site à site, robuste, ports UDP 500/4500
SSL/TLS  → client à site, passe les firewalls (port 443)
WireGuard → moderne, rapide, UDP 51820
PPTP     → ❌ OBSOLÈTE, ne jamais utiliser

IPSET PHASES
Phase 1 (IKE) → authentification + négociation algorithmes
Phase 2       → chiffrement des données

ATTAQUES COURANTES
DoS/DDoS       → saturer un service
MITM           → interception/modification du trafic
ARP Spoofing   → empoisonnement de la table ARP
Phishing       → vol de credentials par email
Brute Force    → tester toutes les combinaisons de MDP
Ransomware     → chiffrement des données contre rançon

CONTRE-MESURES CLÉS
MFA            → indispensable pour VPN et comptes privilégiés
Segmentation   → VLAN + DMZ pour limiter la propagation
Journalisation → tracer pour détecter et investiguer
Mises à jour   → patch management régulier
DAI            → Dynamic ARP Inspection contre ARP Spoofing
IPS            → bloquer les attaques en temps réel

RÉPONSE À INCIDENT (ordre)
1. Contenir  → désactiver les comptes, isoler les machines
2. Analyser  → logs, forensic
3. Corriger  → patcher, reconfigurer
4. Prévenir  → sensibilisation, nouvelles règles
```

---

> ✅ **À retenir** : La sécurité n'est pas un produit qu'on installe — c'est une **démarche continue**. Un firewall seul ne suffit pas. C'est la combinaison firewall + IPS + MFA + segmentation + journalisation + sensibilisation qui constitue une vraie défense en profondeur.
> 
> 🔑 **Le maillon faible est presque toujours humain** : phishing, mot de passe faible, configuration par défaut laissée telle quelle. La technique ne sert à rien sans la sensibilisation.