## Mise en situation professionnelle

---

## 📋 CONTEXTE

Tu viens d'être embauché en tant que **Technicien Systèmes et Réseaux** dans l'entreprise **SWEETCAKES**, une PME spécialisée dans la pâtisserie industrielle. L'entreprise compte 3 services :

- **Direction** (2 personnes)
- **Comptabilité** (3 personnes)
- **Production** (5 personnes)

Tu interviens sur l'infrastructure informatique suite au départ d'un collaborateur et à l'arrivée d'un nouveau.

---

## 🖥️ INFRASTRUCTURE À MONTER

### Machines virtuelles (VirtualBox)

| Nom          | OS                  | Fonction         | RAM  | Réseau                  |
| ------------ | ------------------- | ---------------- | ---- | ----------------------- |
| **SRVWIN01** | Windows Server 2022 | AD-DS, DNS, DHCP | 4 Go | Réseau interne "intnet" |
| **SRVLX01**  | Debian 12           | Serveur Linux    | 2 Go | Réseau interne "intnet" |
| **CLIENT01** | Windows 10/11       | Poste client 1   | 2 Go | Réseau interne "intnet" |
| **CLIENT02** | Windows 10/11       | Poste client 2   | 2 Go | Réseau interne "intnet" |
| **UBUNTU-CLIENT** | Ubuntu Desktop | Client Linux SSH | 2 Go | Réseau interne "intnet" |
| **IPBX** | FreePBX (Linux) | Serveur téléphonie VoIP | 2 Go | Réseau interne "intnet" |

> ⚠️ **UBUNTU-CLIENT** n'est pas intégré au domaine Windows. Il est utilisé **uniquement dans la Partie 10 (SSH)** pour simuler un vrai échange client/serveur entre deux machines distinctes.


### Plan d'adressage IP

| Machine  | Adresse IP   | Masque              | Passerelle   | DNS          |
| -------- | ------------ | ------------------- | ------------ | ------------ |
| SRVWIN01 | 172.16.10.10 | /24 (255.255.255.0) | -            | 127.0.0.1    |
| SRVLX01  | 172.16.10.20 | /24 (255.255.255.0) | 172.16.10.10 | 172.16.10.10 |
| CLIENT01 | DHCP         | /24                 | Via DHCP     | Via DHCP     |
| CLIENT02 | DHCP         | /24                 | Via DHCP     | Via DHCP     |
| **UBUNTU-CLIENT** | **172.16.10.30** | /24 (255.255.255.0) | 172.16.10.10 | 172.16.10.10 |
| **IPBX** | **172.16.10.40** | /24 (255.255.255.0) | 172.16.10.10 | 172.16.10.10 |

### Informations du domaine

- **Nom de domaine** : sweetcakes.lan
- **Nom NetBIOS** : SWEETCAKES
- **Mot de passe Administrateur** : Azerty1*

---

## ⚠️ LÉGENDE DES RÉPONSES

- **🖱️ GUI** = Solution graphique (interface Windows)
- **⌨️ PowerShell** = Solution en ligne de commande (bonus)

---

# PARTIE 1 : INSTALLATION ET CONFIGURATION DE BASE

> [!warning] ⚠️ ORDRE D'EXÉCUTION RECOMMANDÉ
> 1.1 → 1.2 → 1.3 → **1.4 (AD-DS)** → **PARTIE 3 (DHCP)** → **1.5 (vérif IP)** → **PARTIE 2 (DNS)** → **1.6 (jointure CLIENT01)** → **1.7 (jointure CLIENT02)**
> ⚠️ AD-DS (1.4) doit être installé AVANT le DHCP (Partie 3) : l'étape "Complete DHCP Configuration" exige qu'Active Directory soit opérationnel.
> ⚠️ Le DHCP (Partie 3) doit être configuré AVANT de joindre les clients (1.5 / 1.6) : les clients doivent recevoir une IP automatiquement dans la plage 110-150 avant de rejoindre le domaine.
> ℹ️ Le DNS (Partie 2) peut être configuré après que les clients ont rejoint le domaine.

---

## Exercice 1.1 - Configuration réseau du serveur Windows

**Consigne :** Sur **SRVWIN01**, configure une adresse IP statique selon le plan d'adressage.

> [!tip]- Réponse
> 
> ### 🖱️ Solution graphique (GUI)
> 
> **Étape 1 :** Ouvrir le Network and Sharing Center
> 
> - Clic droit sur l'icône réseau en bas à droite (barre des tâches)
> - Cliquer sur **"Open Network & Internet settings"**
> - Cliquer sur **"Network and Sharing Center"** (en bas)
> 
> **Étape 2 :** Accéder aux propriétés de la carte réseau
> 
> - Cliquer sur **"Change adapter settings"** (menu de gauche)
> - Clic droit sur **"Ethernet"** → **"Properties"**
> 
> **Étape 3 :** Configurer IPv4
> 
> - Double-cliquer sur **"Internet Protocol Version 4 (TCP/IPv4)"**
> - Cocher **"Use the following IP address"**
> - Remplir :
>     - **Adresse IP :** 172.16.10.10
>     - **Subnet mask:** 255.255.255.0
>     - **Passerelle par défaut :** (laisser vide)
> - Cocher **"Use the following DNS server address"**
>     - **DNS Server préféré :** 127.0.0.1
> - Cliquer **OK** puis **Fermer**
> 
> **Étape 4 :** Vérifier la configuration
> 
> - Ouvrir **cmd** (touche Windows + R, taper `cmd`, Entrée)
> - Taper : `ipconfig /all`
> - Vérifier que l'IP est bien 172.16.10.10
> 
> ---
> 

---

## Exercice 1.2 - Renommer le serveur Windows

**Consigne :** Sur **SRVWIN01**, renomme la machine en **SRVWIN01**.

> [!tip]- Réponse
> 
> ### 🖱️ Solution graphique (GUI)
> 
> - Clic droit sur **"This PC"** → **"Properties"**
> - Cliquer sur **"Rename this PC"**
> - Taper : **SRVWIN01** → **OK** → **Restart maintenant**
> - Verify: `hostname` → SRVWIN01
> 

---

## Exercice 1.3 - Préparer la connectivité client avant AD (IP statique + fichier hosts)

**SCÉNARIO :** Avant d'installer AD-DS et DNS, le client ne peut pas résoudre le nom du serveur.
**Consigne :** Sur **CLIENT01**, configure une IP statique temporaire et fais en sorte que **SRVWIN01** soit joignable par son nom avant l'installation du DNS.

> [!tip]- Réponse
> 
> **Étape 1 :** IP statique temporaire sur CLIENT01
> - TCP/IPv4 → IP : 172.16.10.50 | Masque : 255.255.255.0 | Passerelle : 172.16.10.10 | DNS : 172.16.10.10
> 
> **Étape 2 :** Notepad (Run as administrator) → `C:\Windows\System32\drivers\etc\hosts`
> ```
> 172.16.10.10    SRVWIN01
> 172.16.10.10    srvwin01.sweetcakes.lan
> ```
> 
> **Étape 3 :** `ping SRVWIN01` → doit répondre 172.16.10.10
> 
> **⚠️ Note :** Config temporaire. Après DHCP installé → repasser en DHCP, le fichier hosts ne sera plus nécessaire.

---

## Exercice 1.4 - Installation du rôle AD-DS

**Consigne :** Sur **SRVWIN01**, installe le rôle **AD-DS** et promeut le serveur en contrôleur de domaine pour le domaine **sweetcakes.lan**.

> [!tip]- Réponse
> 
> **Partie A - Installer le rôle AD-DS**
> - Server Manager → Manage → Add Roles and Features → **Active Directory Domain Services (AD DS)** → Install
> 
> **PART B - Promote to Domain Controller**
> - Drapeau jaune ⚠️ → **"Promote this server to a domain controller"**
> - **Add a new forest** → sweetcakes.lan
> - Mot de passe DSRM : **Azerty1***
> - Nom NetBIOS : **SWEETCAKES**
> - Install → le serveur redémarre automatiquement
> - Log in with: **SWEETCAKES\Administrateur** / Azerty1*
> - Verify: `systeminfo | findstr Domain` → sweetcakes.lan
> 

---

## Exercice 1.5 - Vérifier que CLIENT01 reçoit une IP via DHCP

**⚠️ PRÉREQUIS :** La Partie 3 (DHCP) doit être complétée avant cet exercice.

**Consigne :** Sur **CLIENT01**, repasse en DHCP et vérifie que l'IP reçue est dans la bonne plage.

---

## Exercice 1.6 - Joindre CLIENT01 au domaine

**⚠️ PRÉREQUIS :** La Partie 2 (DNS) doit être complétée avant cet exercice.

**Consigne :** Sur **CLIENT01**, joins la machine au domaine **sweetcakes.lan**.

> [!tip]- Réponse
> 
> **Étape 1-3 :** TCP/IPv4 → Obtenir une adresse IP automatiquement
> ```cmd
> ipconfig /release
> ipconfig /renew
> ipconfig /all
> ```
> 
> **Étape 4 :** `nslookup sweetcakes.lan` + `ping SRVWIN01` → doit répondre
> 
> **Étape 5 :** This PC → Properties → Change settings → **Domain: sweetcakes.lan**
> - Identifiants : **Administrateur** / **Azerty1*** → Redémarrer
> - Verify: `systeminfo | findstr Domain` → sweetcakes.lan
> 

---

## Exercice 1.7 - Joindre CLIENT02 au domaine

**Consigne :** Sur **CLIENT02**, joins la machine au domaine **sweetcakes.lan**.

> [!tip]- Réponse
> Procédure identique à l'exercice 1.5 appliquée sur CLIENT02.
> - DHCP → `ipconfig /release` → `ipconfig /renew`
> - Vérifier IP dans la plage 172.16.10.**111-150** (110 réservé CLIENT01, 151-200 exclu)
> - `nslookup sweetcakes.lan` et `ping SRVWIN01`
> - Rejoindre le domaine sweetcakes.lan → Redémarrer
> - `systeminfo | findstr Domain` → sweetcakes.lan
> 
> **⚠️ CLIENT02 sert principalement aux tests de téléphonie** (2ème softphone).

---

# PARTIE 2 : DNS

---

## Exercice 2.1 - Ajouter une entrée dans le fichier hosts

**Consigne :** Sur **CLIENT01**, fais en sorte que le nom `serveur-test` pointe vers **172.16.10.99** sans passer par le serveur DNS.

> [!tip]- Réponse
> - Notepad (exécuter en administrateur) → `C:\Windows\System32\drivers\etc\hosts`
> - Add: `172.16.10.99 serveur-test`
> - Enregistrer → Tester : `ping serveur-test`
>
> Le fichier **hosts** est consulté **avant** le DNS. C'est une résolution locale, manuelle — utile pour les tests ou pour forcer une résolution sans toucher au serveur DNS.

---

## Exercice 2.2 - Créer un alias DNS (CNAME)

**Consigne :** Sur **SRVWIN01**, crée un enregistrement **A** pour `SRVLX01` (172.16.10.20) et un alias **CNAME** `intranet` pointant vers `SRVLX01`.

> [!tip]- Réponse
> - **DNS Manager** → sweetcakes.lan → Right-click → **New Host (A)** → Name: `SRVLX01` / IP: `172.16.10.20` → **Add Host**
> - Right-click on `sweetcakes.lan` → **New Alias (CNAME)** → Alias name: `intranet` → FQDN of target host: `SRVLX01.sweetcakes.lan` → **OK**
> - `nslookup intranet.sweetcakes.lan` → 172.16.10.20
>
> Un enregistrement **CNAME** crée un alias pointant vers un autre nom DNS. Si l'IP du serveur change, on ne modifie que le A — le CNAME suit automatiquement. Utile pour exposer un même serveur sous plusieurs noms (`intranet`, `www`, `web`...).

---

## Exercice 2.3 - Créer un enregistrement DNS A pour l'IPBX

**Consigne :** Sur **SRVWIN01**, crée un enregistrement **A** pour `ipbx` (172.16.10.40).

> [!tip]- Réponse
> - **DNS Manager** → sweetcakes.lan → Right-click → **New Host (A)** → Name: `ipbx` / IP: `172.16.10.40` → **Add Host**
> - `nslookup ipbx.sweetcakes.lan` + `ping ipbx.sweetcakes.lan`
>
> Un enregistrement **A** associe un nom à une adresse IPv4. C'est l'enregistrement DNS de base — sans lui, aucun service (IPBX, mail, web...) ne peut être résolu par son nom sur le réseau.
> 

---

## Exercice 2.4 - Créer un enregistrement MX fictif

**Consigne :** Sur **SRVWIN01**, crée un enregistrement **A** pour `mail` (172.16.10.25) et déclare-le comme serveur de messagerie du domaine avec une priorité de **10**.

> [!tip]- Réponse
> - **DNS Manager** → sweetcakes.lan → Right-click → **New Host (A)** → Name: `mail` / IP: `172.16.10.25` → **Add Host**
> - Right-click on `sweetcakes.lan` → **New Mail Exchanger (MX)** → Host or child domain: laisser vide → FQDN of mail server: `mail.sweetcakes.lan` → Priority: `10` → **OK**
> - `nslookup -type=MX sweetcakes.lan` → mail exchanger = 10 mail.sweetcakes.lan
>
> Un enregistrement **MX** indique quel serveur reçoit les emails pour le domaine. La priorité détermine l'ordre en cas de plusieurs serveurs mail : **valeur la plus basse = priorité la plus haute**.

---

## Exercice 2.5 - Comprendre l'enregistrement SOA

**Consigne :** Interroge le serveur DNS pour afficher l'enregistrement **SOA** de la zone `sweetcakes.lan` et explique chaque champ retourné.

> [!tip]- Réponse
> `nslookup -type=SOA sweetcakes.lan`
> 
> | Champ | Signification |
> |-------|---------------|
> | Primary server | Primary DC of the zone |
> | Numéro de série | Incrémenté à chaque modif → synchro DNS secondaires |
> | Actualisation | Fréquence de vérification par les secondaires |
> | Nouvelle tentative | Délai de réessai si refresh échoue |
> | Expiration | Delay before secondary stops answering |
> | TTL minimum | Durée de cache par défaut |
> 
> **📌 Le SOA est créé automatiquement** avec la zone.
>
> Le **SOA (Start of Authority)** est l'enregistrement qui prouve qu'un serveur DNS fait autorité sur une zone. Il est unique par zone et contient toutes les infos de synchronisation avec les serveurs DNS secondaires.

---

## Exercice 2.6 - Créer un enregistrement NS fictif

**Consigne :** Sur **SRVWIN01**, crée un enregistrement **A** pour `ns2` (172.16.10.11) et déclare-le comme serveur de noms secondaire de la zone.

> [!tip]- Réponse
> - **DNS Manager** → sweetcakes.lan → Right-click → **New Host (A)** → Name: `ns2` / IP: `172.16.10.11` → **Add Host**
> - Right-click on `sweetcakes.lan` → **Properties** → **Name Servers** tab → **Add** → FQDN: `ns2.sweetcakes.lan` → **Resolve** → **OK**
> - `nslookup -type=NS sweetcakes.lan`

Un enregistrement **NS (Name Server)** indique **quels serveurs font autorité** pour la zone `sweetcakes.lan`.

**Concrètement ici :**

- `ns1.sweetcakes.lan` = le serveur DNS principal (déjà là)
- `ns2.sweetcakes.lan` = un **deuxième serveur DNS** (redondance)

**Pourquoi c'est important :**

- Si `ns1` tombe en panne → `ns2` prend le relais
- Les clients DNS savent qu'ils peuvent interroger l'un **ou** l'autre
- C'est une bonne pratique en entreprise : **toujours au moins 2 serveurs DNS**

À l'oral le jury peut te demander : _"Pourquoi avoir deux enregistrements NS ?"_ → tu réponds **haute disponibilité / tolérance aux pannes**

---

## Exercice 2.7 - Créer un enregistrement AAAA (IPv6)

**Consigne :** Sur **SRVWIN01**, crée un enregistrement **AAAA** associant `srvlx01-v6` à l'adresse **fd00::20**.

> [!tip]- Réponse
> - **DNS Manager** → sweetcakes.lan → Right-click → **New Host (A or AAAA)** → Name: `srvlx01-v6` / IP: `fd00::20` → **Add Host**
> - `nslookup -type=AAAA srvlx01-v6.sweetcakes.lan`
>
> Un enregistrement **AAAA** est l'équivalent IPv6 du A : il associe un nom à une adresse IPv6. Le "quad A" vient des 4 × 32 bits = 128 bits d'une adresse IPv6, contre 32 bits pour l'enregistrement A (IPv4).

---

## Exercice 2.8 - Créer une zone de recherche inversée DNS (PTR)

**Consigne :** Sur **SRVWIN01**, crée la zone de recherche inversée pour **172.16.10.0/24** et ajoute les enregistrements **PTR** pour SRVWIN01 et SRVLX01.

> [!tip]- Réponse
>
> **Étape 1 — Créer la zone de recherche inversée :**
> DNS Manager → Right-click **Reverse Lookup Zones** → **New Zone** → Next → **Primary Zone** → Next → **IPv4 Reverse Lookup Zone** → Next → Network ID: `172.16.10` → Next → Next → **Finish**
>
> **Étape 2 — Créer les enregistrements PTR :**
> Right-click on `10.16.172.in-addr.arpa` → **New Pointer (PTR)**
>
> | Host IP Address | Host name |
> |-----------------|-----------|
> | `172.16.10.10` | `SRVWIN01.sweetcakes.lan` |
> | `172.16.10.20` | `SRVLX01.sweetcakes.lan` |
>
> **Étape 3 — Tester depuis CLIENT01 :**
> ```
> nslookup 172.16.10.10   → SRVWIN01.sweetcakes.lan
> nslookup 172.16.10.20   → SRVLX01.sweetcakes.lan
> ```
>
> Un enregistrement **PTR** fait la résolution DNS inversée : IP → Nom. Il est utilisé par les serveurs mail pour vérifier l'identité de l'expéditeur (anti-spam) et par les outils de diagnostic comme `nslookup` et `traceroute`.

---

## 📌 Récap types d'enregistrements DNS

| Type | Rôle | Exemple |
|------|------|---------|
| **A** | Nom → IPv4 | srvlx01 → 172.16.10.20 |
| **AAAA** | Nom → IPv6 | srvlx01-v6 → fd00::20 |
| **CNAME** | Alias → un autre nom | intranet → srvlx01 |
| **MX** | Serveur de messagerie | sweetcakes.lan → mail.sweetcakes.lan (10) |
| **NS** | Serveurs DNS faisant autorité | sweetcakes.lan → srvwin01.sweetcakes.lan |
| **PTR** | IP → Nom (résolution inversée) | 172.16.10.10 → srvwin01.sweetcakes.lan |
| **SOA** | Infos d'autorité de la zone (auto) | Serveur primaire, serial, refresh... |

# PARTIE 3 : DHCP

---

## Exercice 3.1 - Installation et configuration du rôle DHCP

**Consigne :** Sur **SRVWIN01**, installe le rôle **DHCP** et crée l'étendue **LAN_SWEETCAKES** (172.16.10.100–200, bail 8 jours, passerelle 172.16.10.10, DNS 172.16.10.10).

Résultat attendu : l'étendue **LAN_SWEETCAKES** est Active in the **DHCP Console**. Les IPs distribuées seront dans la plage **172.16.10.110-150** après les exclusions des exercices 3.2 et 3.5.

> [!tip]- Réponse
> 
> **Partie A - Installation**
> - Server Manager → Add Roles and Features → **DHCP Server** → Install
> - Yellow flag → **Terminer la configuration DHCP** → Valider
> 
> **Partie B - Créer l'étendue**
> - DHCP Console → IPv4 → New Scope → LAN_SWEETCAKES
> - Plage : 100-200 | Masque : 255.255.255.0 | Bail : 8 jours
> - Routeur : 172.16.10.10 | DNS : 172.16.10.10
> 

---

## Exercice 3.2 - Ajouter une plage d'exclusion DHCP IPv4

**Consigne :** Sur **SRVWIN01**, ajoute une exclusion DHCP de **172.16.10.100 à 172.16.10.109** dans l'étendue **LAN_SWEETCAKES**.

Résultat attendu : la plage 100-109 apparaît dans **Address Pool** sous la mention « Exclusion Range ».

> [!tip]- Réponse
> - DHCP Console → Address Pool → New Exclusion Range → 100-109
> 

---

## Exercice 3.3 - Ajouter une plage d'exclusion DHCP IPv6

**Consigne :** Sur **SRVWIN01**, crée une étendue DHCP **IPv6** (préfixe fd00::/64) et exclure la plage **fd00::1 à fd00::10**.

Résultat attendu : l'étendue IPv6 est Active dans la console et l'exclusion fd00::1-fd00::10 est visible dans Address Pool.

> [!tip]- Réponse
> - DHCP Console → IPv6 → New Scope → prefix fd00::
> - Exclusions → fd00::1 à fd00::10
> 

---

## Exercice 3.4 - Créer une réservation DHCP IPv4

**Consigne :** Sur **SRVWIN01**, crée une réservation DHCP pour que **CLIENT01** reçoive toujours l'adresse **172.16.10.110**.

> [!tip]- Réponse
> - `ipconfig /all` sur CLIENT01 → noter l'adresse MAC
> - DHCP Console → Reservations → New Reservation
>   - Name: CLIENT01 | IP : 172.16.10.110 | MAC : sans tirets
> - `ipconfig /release` puis `ipconfig /renew` → vérifier IP = 172.16.10.110
> 
> **⚠️ PIÈGE :** L'IP de réservation (110) doit être hors de la plage d'exclusion (100-109) ✅

---

## Exercice 3.4b - Créer une réservation DHCP IPv6

**Consigne :** Sur **SRVWIN01**, crée une réservation DHCP **IPv6** pour **CLIENT01** avec l'adresse **fd00::100**.

> [!tip]- Réponse
> - `ipconfig /all` → noter le **DUID DHCPv6 client**
> - DHCP Console → IPv6 → Reservations → New Reservation → fd00::100

---

## Exercice 3.5 - Modifier la plage DHCP IPv4

**Consigne :** Sur **SRVWIN01**, ajoute une seconde exclusion DHCP de **172.16.10.151 à 172.16.10.200** dans l'étendue **LAN_SWEETCAKES**.

> [!tip]- Réponse
> - Address Pool → New Exclusion Range → 151-200
> - Résultat : exclusions 100-109 + 151-200 → distribution : **110 à 150**

---

## Exercice 3.5b - Modifier la plage DHCP IPv6

**Consigne :** Sur **SRVWIN01**, ajoute une exclusion DHCP IPv6 de **fd00::101 à fd00::200**.

⚠️ L'exclusion démarre à `fd00::101` (et non `fd00::100`) car `fd00::100` est déjà réservé pour CLIENT01 en exercice 3.4b. Inclure `fd00::100` dans l'exclusion créerait une incohérence in the **DHCP Console**.

Résultat attendu : l'exclusion fd00::101-fd00::200 est visible in the **DHCP Console** IPv6.

> [!tip]- Réponse
> - IPv6 → Exclusions → fd00::101 à fd00::200

---

# PARTIE 4 : GESTION DES UTILISATEURS ET GROUPES AD

---

## Exercice 4.1 - Créer la structure d'OU

**Consigne :** Sur **SRVWIN01**, crée la structure d'OU suivante dans Active Directory :

```
sweetcakes.lan
├── OU=Utilisateurs
│   ├── OU=Direction
│   ├── OU=Comptabilite
│   └── OU=Production
├── OU=Ordinateurs
└── OU=Groupes
```

> [!tip]- Réponse
> - **dsa.msc** (Active Directory Users and Computers) → right-click sweetcakes.lan → New → Organizational Unit → **Utilisateurs**
> - Right-click Utilisateurs → New → Organizational Unit → Direction / Comptabilite / Production
> - Répéter pour Ordinateurs et Groupes
> 

---

## Exercice 4.2 - Créer les groupes de sécurité

**Consigne :** Sur **SRVWIN01**, crée les 3 groupes de sécurité **Global** suivants dans l'OU **Groupes** : `GRP_Direction`, `GRP_Comptabilite`, `GRP_Production`.

Vérifie que les 3 groupes apparaissent bien dans `OU=Groupes,DC=sweetcakes,DC=lan`.

> [!tip]- Réponse
> - **dsa.msc** → OU Groupes → New → Group → Scope: **Global** | Type: **Security**
> 

---

## Exercice 4.3 - Créer les utilisateurs

**Consigne :** Sur **SRVWIN01**, crée les 4 comptes suivants dans leur OU respective (MDP: **Azerty123!**, n'expire jamais) :

| Prénom | Nom | Login | OU |
|--------|-----|-------|----|
| Marie | DUPONT | m.dupont | Direction |
| Jean | MARTIN | j.martin | Comptabilite |
| Pierre | DURAND | p.durand | Comptabilite |
| Sophie | BERNARD | s.bernard | Production |

> [!tip]- Réponse
> - **dsa.msc** (Active Directory Users and Computers) → OU correspondante → New → User → remplir les champs
> - MDP : Azerty123! | Décocher "User must change password at next logon" | Cocher "Password never expires"
> 

---

## Exercice 4.4 - Ajouter les utilisateurs aux groupes

**Consigne :** Sur **SRVWIN01**, ajoute chaque utilisateur dans son groupe : m.dupont → GRP_Direction, j.martin et p.durand → GRP_Comptabilite, s.bernard → GRP_Production.

> [!tip]- Réponse
> - GRP_Direction → m.dupont | GRP_Comptabilite → j.martin, p.durand | GRP_Production → s.bernard
> 

---

## Exercice 4.5 - Mettre en place l'AGDLP

**Consigne :** Sur **SRVWIN01**, crée les groupes **Domain Local** `DL_Projets_RW` et `DL_Projets_RO` dans l'OU Groupes, puis ajoute GRP_Comptabilite dans RW et GRP_Production dans RO.

> [!tip]- Réponse
> - New Group → DL_Projets_RW | Scope: **Domain Local** | Type: Security
> - Répéter pour DL_Projets_RO
> - DL_Projets_RW → Members → Add → GRP_Comptabilite
> - DL_Projets_RO → Members → Add → GRP_Production
> 
> **Schéma AGDLP :**
> ```
> p.durand (Account) → GRP_Comptabilite (Global) → DL_Projets_RW (Domain Local) → Modify sur C:\Projets
> s.bernard (Account) → GRP_Production (Global) → DL_Projets_RO (Domain Local) → Read sur C:\Projets
> ```
> 
> **⚠️ PATCH — Correction schéma :** l.moreau n'existe pas encore à cette étape (créée en Partie 5). Le schéma utilise p.durand comme représentant de la Comptabilité.
> 
> **⚠️ Pourquoi AGDLP ?**
> - NTFS permissions ALWAYS on **Domain Local** groups (DL_)
> - Users ALWAYS in **Global** groups (GRP_)
> - Benefit: new department → just add its GRP_ to the DL_ without touching NTFS permissions

---

# PARTIE 5 : SCÉNARIO - DÉPART ET ARRIVÉE D'UN COLLABORATEUR

---

## Exercice 5.1 - Départ d'un collaborateur

**SCÉNARIO :** Jean MARTIN quitte l'entreprise.
**Consigne :** Sur **SRVWIN01**, applique la procédure de départ pour **j.martin** (Jean MARTIN) : désactive le compte, réinitialise son mot de passe à **Desactive2024!**, retire-le de **GRP_Comptabilite** et déplace-le dans une OU **Utilisateurs_Desactives**.

> [!tip]- Réponse
> 
> - Créer **OU Utilisateurs_Desactives**
> - Right-click Jean MARTIN → **Disable Account**
> - Right-click → **Reset Password** → Desactive2024!
> - Double-clic → onglet **Member Of** → retirer GRP_Comptabilite (⚠️ garder **Domain Users**)
> - Right-click → **Move** → Utilisateurs_Desactives
> 

---

## Exercice 5.2 - Arrivée d'un nouveau collaborateur

**SCÉNARIO :** Lucie MOREAU remplace Jean MARTIN au service Comptabilité.
**Consigne :** Sur **SRVWIN01**, crée le compte **l.moreau** (Lucie MOREAU) dans l'OU Comptabilite en reprenant les attributs professionnels de **j.martin**, et ajoute-la au groupe **GRP_Comptabilite**.

> [!tip]- Réponse
> 
> - Consulter les attributs de j.martin (onglet **Organization**)
> - Créer dans Utilisateurs → Comptabilite : l.moreau / Azerty123!
> - Department: Comptabilite | Company: SWEETCAKES
> - GRP_Comptabilite → Members → Add → l.moreau
> 

---

# PARTIE 6 : SERVEUR DE FICHIERS ET PARTAGES

---

## Exercice 6.1 - Install le rôle File Server

**Consigne :** Sur **SRVWIN01**, installe le rôle **File Server**.

Résultat attendu : le rôle File Server apparaît dans le Server Manager avec l'état « Installé ».

> [!tip]- Réponse
> - Server Manager → Add Roles and Features → File and Storage Services → **File Server** → Install
> - Terminer via "Complete DHCP Configuration" si demandé
> 

---

## Exercice 6.2 - Créer les dossiers partagés

**Consigne :** Sur **SRVWIN01**, crée et partage les 3 dossiers suivants avec les permissions indiquées :

| Dossier | Chemin | Partage | NTFS |
|---------|--------|---------|------|
| Partage | C:\Partage | Everyone → Full Control | Utilisateurs du domaine → Modification |
| Wallpapers | C:\Wallpapers | Everyone → Read | Utilisateurs du domaine → Lecture |
| Projets | C:\Projets | Everyone → Full Control | Configuré en AGDLP Partie 8 |

> [!tip]- Réponse
> - Créer le dossier → clic droit → Properties → Sharing → Advanced Sharing → cocher **"Share this folder"**
> - Permissions → Everyone → Full Control (or Read, depending on the folder)
> - Security tab → Add **Domain Users** → corresponding rights
> 
> **⚠️ IMPORTANT :** Copier une image **compta.jpg** dans C:\Wallpapers maintenant — nécessaire pour la GPO fond d'écran (Partie 8). Si absent → la GPO ne fonctionnera pas.
> 
> **⚠️ Best practice:** Broad permissions **on the share** (Everyone Full Control), fine-grained filtering **via NTFS**

---

# PARTIE 7 : ADAC — POLITIQUE DE MOT DE PASSE

---

## Exercice 7.1 - Ouvrir ADAC

**Consigne :** Sur **SRVWIN01**, ouvre l'**Active Directory Administrative Center (ADAC)** et explique sa différence avec la console classique **dsa.msc**.

> [!tip]- Réponse
> - `dsac.exe` ou Server Manager → Tools → **Active Directory Administrative Center (ADAC)**
> 
> | Outil | Commande | Usage |
> |-------|----------|-------|
> | **ADUC** | dsa.msc | Day-to-day users/groups/OU management |
> | **ADAC** | dsac.exe | FGPP, AD Recycle Bin, advanced views |
> | **GPMC** | gpmc.msc | Gestion des GPO |

---

## Exercice 7.2 - Créer une PSO (FGPP)

**Consigne :** Sur **SRVWIN01**, dans **ADAC**, crée la politique de mot de passe **PSO_Direction** (priorité 10, 12 caractères min, 60 jours, 5 tentatives, 30 min verrouillage, historique 10) et applique-la au groupe **GRP_Direction**.

Navigation : ADAC → sweetcakes (local) → System → **Password Settings Container** → New → Password Settings.

> [!tip]- Réponse
> - ADAC → System → **Password Settings Container** → New → Password Settings
> - Name: PSO_Direction | Precedence: **10** | Min length: 12 | Max age: 60 jours | Lockout threshold: 5 | Lockout duration: 30 min | Password history: 10
> - Applies directly to → Add → **GRP_Direction**
> 
> **📌 À retenir :**
> - PSO applies to a **Global group** or a **user**, never directly to an OU
> - **Priorité** : chiffre le plus **bas** = priorité la plus haute en cas de conflit
> - Niveau fonctionnel minimum : **Windows Server 2008**
> 

---

# PARTIE 8 : GPO (STRATÉGIES DE GROUPE)

---

## Exercice 8.0a - GPO — Bloquer le Panneau de configuration

**Consigne :** Sur **SRVWIN01**, crée la GPO **GPO_Bloquer_PanneauConfig** liée à l'OU **Utilisateurs** et bloque l'accès au Panneau de configuration pour les utilisateurs du domaine.

> [!tip]- Réponse
> - **gpmc.msc** (Group Policy Management Console) → New → GPO_Bloquer_PanneauConfig → Link to OU Utilisateurs
> - Edit → User Configuration → Policies → Administrative Templates → **Control Panel**
> - **"Prohibit access to Control Panel and PC settings"** → **Enabled**
> - Delegation tab → Advanced → Add **Domain Computers** → Read (only) (MS16-072)
> ⚠️ **Oral :** Authenticated Users inclut déjà Domain Computers — l'ajout en Delegation est redondant ici, mais défensif. Obligatoire uniquement quand Authenticated Users est **supprimé** du Security Filtering.

---

## Exercice 8.0b - GPO — Bloquer l'invite de commandes (CMD)

**Consigne :** Sur **SRVWIN01**, crée la GPO **GPO_Bloquer_CMD** liée à l'OU **Utilisateurs** et bloque l'accès à l'invite de commandes pour les utilisateurs du domaine.

> [!tip]- Réponse
> - **gpmc.msc** → New → GPO_Bloquer_CMD → Link to OU Utilisateurs
> - Edit → User Configuration → Policies → Administrative Templates → System
> - **"Prevent access to the command prompt"** → **Enabled**
> - Option **"Also disable the command prompt script processing?"** → **No** ← si Yes, les scripts de connexion ne fonctionnent plus
> - Scope tab → Security Filtering : keep **Authenticated Users** (applies to all domain users)
> - Onglet Delegation → Advanced → Ajouter **Domain Computers** → **Read** ← requis depuis le patch MS16-072

---

## Exercice 8.1 - GPO de fond d'écran avec filtrage de sécurité

### 8.1a — Préparer le wallpaper

**Consigne :** Sur **SRVWIN01**, copie une image **compta.jpg** dans `C:\Wallpapers`.

> [!tip]- Réponse
> - Vérifier que **compta.jpg** est dans `\\SRVWIN01\Wallpapers`
> - Si absent → copier une image .jpg dans C:\Wallpapers sur SRVWIN01
> - NTFS permissions: **Domain Users** → **Read**

---

### 8.1b — Créer la GPO avec filtrage de sécurité

**Consigne :** Sur **SRVWIN01**, crée la GPO **GPO_Compta_Wallpaper** liée à l'OU **Comptabilite**, qui impose le fond d'écran `\\SRVWIN01\Wallpapers\compta.jpg` uniquement aux membres de **GRP_Comptabilite**.

> [!tip]- Réponse
> 
> **Étape 1 :** Créer GPO_Compta_Wallpaper
> 
> **Étape 2 :** Link to OU Comptabilite
> 
> **Étape 3 :** Edit → User Configuration → Policies → Administrative Templates → Desktop → Desktop
> - **"Desktop Wallpaper"** → Enabled → `\\SRVWIN01\Wallpapers\compta.jpg` → Style: Stretch
> 
> **Étape 4 :** Security Filtering
> - Onglet Scope → Security Filtering → Supprimer **Authenticated Users**
> - Add → **GRP_Comptabilite**
> 
> **Étape 5 :** Delegation tab → Advanced
> - Ajouter **GRP_Comptabilite** → Read + Apply Group Policy
> 
> **Étape 6 :** Ajouter **Domain Computers** → **Read** (requis MS16-072)
> - Delegation tab → Advanced → Add **Domain Computers** → Read (only)
> - ⚠️ Sans Domain Computers en Read → la GPO ne s'applique pas depuis le patch MS16-072

---

### 8.1c — Tester la GPO

**Consigne :** Sur **CLIENT01**, vérifie que la GPO **GPO_Compta_Wallpaper** s'applique correctement au compte **l.moreau**.

> [!tip]- Réponse
> - Sur CLIENT01 : `gpupdate /force` → déconnexion / reconnexion avec un compte Comptabilite
> - Wallpaper must change
> - `gpresult /r` → GPO_Compta_Wallpaper doit apparaître

---

## Exercice 8.2 - Restriction horaire de connexion

**Consigne :** Sur **SRVWIN01**, configure les horaires de connexion du compte **s.bernard** pour autoriser uniquement les connexions du lundi au vendredi de 7h à 17h.

> [!tip]- Réponse
> - **dsa.msc** (Active Directory Users and Computers) → Sophie BERNARD → **Account** tab → **Logon Hours...**
> - Deny all (full blue) → select Monday–Friday 07:00–17:00 → set to Logon Permitted (white)
> 
> **⚠️ Note :** Se configure sur le **compte utilisateur**, pas via GPO.

---

## Exercice 8.3 - GPO de mappage de lecteur réseau

### 8.3a — Vérifier le partage

**Consigne :** Depuis **CLIENT01**, vérifie que le partage `\\SRVWIN01\Partage` est accessible avec un compte du domaine.

Test : Explorateur Windows → `\\SRVWIN01\Partage` → doit être accessible ✅

> [!tip]- Réponse
> - `\\SRVWIN01\Partage` → doit être accessible
> - Share: Everyone → Full Control | NTFS: **Domain Users** → Modify

---

### 8.3b — Créer la GPO

**Consigne :** Sur **SRVWIN01**, crée la GPO **GPO_Lecteur_Partage** liée à l'OU **Utilisateurs** qui mappe automatiquement le lecteur **S:** vers `\\SRVWIN01\Partage` pour tous les utilisateurs du domaine.

> [!tip]- Réponse
> - New → GPO_Lecteur_Partage → Link to OU Utilisateurs
> - Edit → User Configuration → Preferences → Windows Settings → **Drive Maps**
> - New → Mapped Drive → Action: Create | Location: `\\SRVWIN01\Partage` | Reconnect: ✅ | Drive Letter: **S:**
> - Scope tab → Security Filtering : keep **Authenticated Users** (or **Domain Users**) ← GPO applies to all
> - Onglet Delegation → Advanced → Ajouter **Domain Computers** → **Read** ← requis depuis le patch MS16-072 — this GPO applies to all domain users

---

### 8.3c — Tester

**Consigne :** Depuis **CLIENT01**, vérifie que le lecteur **S:** est automatiquement mappé après connexion avec un compte du domaine.

Résultat attendu : le lecteur **S:** est bien mappé vers `\\SRVWIN01\Partage` ✅

> [!tip]- Réponse
> - `gpupdate /force` → log off / log back in
> - Lecteur **S:** doit apparaître dans l'Explorateur
> - `net use` → `S: \\SRVWIN01\Partage`

---

## Exercice 8.4 - AGDLP sur le partage Projets

### 8.4a — Configurer les permissions NTFS

**Consigne :** Sur **SRVWIN01**, configure les permissions **NTFS** sur `C:\Projets` : DL_Projets_RW → Modify, DL_Projets_RO → Read & Execute.

> [!tip]- Réponse
> - C:\Projets → Security tab → Edit → Add **DL_Projets_RW** → **Modify**
> - Add DL_Projets_RO → Read & Execute
> - ⚠️ Remove the **Users** group from NTFS permissions if present
> 
> **Schéma AGDLP final :**
> ```
> p.durand / l.moreau → GRP_Comptabilite (Global) → DL_Projets_RW (DL) → Modify C:\Projets
> s.bernard → GRP_Production (Global) → DL_Projets_RO (DL) → Read C:\Projets
> ```

---

### 8.4b — Tester l'accès

**Consigne :** Depuis **CLIENT01**, vérifie les droits NTFS sur `\\SRVWIN01\Projets` avec **p.durand** (doit pouvoir modifier) et **s.bernard** (lecture seule).

> [!tip]- Réponse
> - **p.durand** → `\\SRVWIN01\Projets` → can create/modify ✅
> - **s.bernard** → `\\SRVWIN01\Projets` → read-only, cannot modify ✅

---

# PARTIE 9 : ADMINISTRATION WINDOWS — SSH + SCP

---

## Exercice 9.1 - Configurer OpenSSH Server sur SRVWIN01

**Consigne :** Sur **SRVWIN01**, installe **OpenSSH Server**, configure-le en démarrage automatique et vérifie la connexion SSH depuis **CLIENT01**.

> [!tip]- Réponse
> 
> **Étape 1-2 :** Settings → Apps → Optional Features → **OpenSSH Server** → Install
> 
> **Étape 3 :** services.msc → **OpenSSH SSH Server** → Startup type: Automatic → Start
> 
> **Étape 4 :** Vérifier règle pare-feu port **22** → wf.msc → Inbound Rules → OpenSSH Server
> 
> **Étape 5 :** Tester : `ssh Administrateur@172.16.10.10` → MDP : Azerty1*
> 
> | Fichier | Chemin |
> |---------|--------|
> | Config serveur | `C:\ProgramData\ssh\sshd_config` |
> | Clés autorisées (admin) | `C:\ProgramData\ssh\administrators_authorized_keys` |
> | Clés autorisées (user) | `C:\Users\<user>\.ssh\authorized_keys` |

---

## Exercice 9.2 - Envoyer une clé publique en SCP vers SRVWIN01

**Consigne :** Depuis **CLIENT01**, configure l'authentification SSH par **clé ed25519** vers **SRVWIN01** (connexion sans mot de passe).

Résultat attendu : `ssh Administrateur@172.16.10.10` depuis CLIENT01 s'ouvre **sans saisir de mot de passe**.

> [!tip]- Réponse
>
> **Étape 1 :** Générer une clé SSH sur CLIENT01
> ```powershell
> ssh-keygen -t ed25519
> ```
>
> **Étape 2 :** Vérifier si le fichier existe déjà sur SRVWIN01 avant de copier
> ```powershell
> ssh Administrateur@172.16.10.10 "type C:\ProgramData\ssh\administrators_authorized_keys"
> ```
> ⚠️ Si le fichier contient déjà des clés → ne pas écraser, ajouter manuellement.
>
> **Étape 3 :** Copier la clé via SCP
> ```powershell
> scp $env:USERPROFILE\.ssh\id_ed25519.pub "Administrateur@172.16.10.10:C:/ProgramData/ssh/administrators_authorized_keys"
> # ⚠️ Utiliser des SLASHS (/) dans le chemin distant, pas des antislashs
> # Les antislashs sont interprétés comme séquences d'échappement par SCP et provoquent une erreur
> ```
> ⚠️ Cette commande **écrase** le fichier existant. Si d'autres clés sont présentes, les ajouter manuellement.
>
> **Étape 4 :** Corriger les permissions (obligatoire)
> ```powershell
> ssh Administrateur@172.16.10.10
> icacls "C:\ProgramData\ssh\administrators_authorized_keys" /inheritance:r /grant "Administrators:F" /grant "SYSTEM:F"
> Restart-Service sshd
> ```
> ⚠️ **Normal :** `Restart-Service sshd` redémarre SSH → la session SSH en cours se coupe immédiatement. C'est attendu — le service redémarre bien. Se reconnecter à l'étape 5.
>
> **Étape 5 :** Tester la connexion sans mot de passe
> ```powershell
> ssh Administrateur@172.16.10.10
> ```
> ⚠️ Pour les comptes **administrateurs** → les clés vont dans `administrators_authorized_keys` (PAS dans ~/.ssh/authorized_keys)

---

## Exercice 9.3 - Connexion Bureau à distance (RDP)

**Consigne :** Active le **Bureau à distance (RDP)** sur **SRVWIN01** et établis une connexion depuis **CLIENT01**.

> [!tip]- Réponse
>
> **Étape 1 — Activer RDP sur SRVWIN01**
> - This PC → Properties → **Remote settings**
> - Cocher **"Allow remote connections to this computer"** → OK
>
> ⚠️ Vérifier que la règle pare-feu **"Remote Desktop"** est active :
> ```cmd
> netsh advfirewall firewall set rule group="Bureau a distance" new enable=yes
> ```
>
> **Étape 2 — Se connecter depuis CLIENT01**
> - Touche Windows → taper **mstsc** → Entrée
> - Ordinateur : **172.16.10.10** (ou **SRVWIN01.sweetcakes.lan**)
> - Utilisateur : **SWEETCAKES\Administrateur** → Mot de passe : Azerty1*
> - Connect → Yes (avertissement certificat)
>

---

# PARTIE 10 : SERVEUR LINUX (SRVLX01)

**⚠️ PRÉREQUIS VM :** Ajouter les disques supplémentaires dans VirtualBox AVANT de démarrer :
- **1 disque de 10 Go** (sdb) pour l'exercice 10.0
- **2 disques de 10 Go** (sdc + sdd) pour les exercices LVM/RAID

---

## Exercice 10.0 - Préparer le disque dur supplémentaire

**Consigne :** Sur **SRVLX01**, partitionne le disque **sdb** en 3 partitions : sdb1 (6 Go, ext4, label DATA, monté sur `/mnt/data`), sdb2 (2 Go, ext4, label PERSO, monté sur `/mnt/perso`), sdb3 (reste, swap).

Rends les montages **permanents** au démarrage via les UUID. Vérifie avec `df -h`, `free -h` et `lsblk -f /dev/sdb`.

> [!tip]- Réponse
> ```bash
> lsblk
> # → Affiche tous les disques et partitions — repère sdb (nouveau disque vide)
> # sda = système (NE PAS TOUCHER) | sdb = nouveau disque
> 
> fdisk /dev/sdb
> # → Lance l'outil de partitionnement interactif sur sdb
> #   Partition 1 : n → p → 1 → Entrée → +6G    (nouvelle partition primaire de 6 Go)
> #   Partition 2 : n → p → 2 → Entrée → +2G    (nouvelle partition primaire de 2 Go)
> #   Partition 3 : n → p → 3 → Entrée → Entrée (prend tout l'espace restant)
> #   Changer type partition 3 : t → 3 → 82      (définit le type comme Linux swap)
> #   w → écrire et quitter                       (écrit la table de partitions sur le disque)
> 
> mkfs.ext4 -L DATA /dev/sdb1
> # → Formate sdb1 en ext4 et lui attribue le label "DATA"
> mkfs.ext4 -L PERSO /dev/sdb2
> # → Formate sdb2 en ext4 et lui attribue le label "PERSO"
> mkswap -L SWAP /dev/sdb3
> # → Prépare sdb3 comme espace d'échange (swap) — ne monte pas, prépare seulement
> 
> mkdir -p /mnt/data /mnt/perso
> # → Crée les deux points de montage (dossiers qui recevront les partitions)
> mount /dev/sdb1 /mnt/data
> # → Monte sdb1 sur /mnt/data — rend la partition accessible comme dossier
> mount /dev/sdb2 /mnt/perso
> # → Monte sdb2 sur /mnt/perso
> swapoff -a
> # → Désactive tous les espaces swap actifs (nécessaire avant d'en activer un nouveau)
> swapon /dev/sdb3
> # → Active sdb3 comme espace swap
> 
> blkid /dev/sdb1 /dev/sdb2 /dev/sdb3
> # → Affiche les UUID de chaque partition — copie ces UUID dans fstab
> nano /etc/fstab
> # → Ouvre le fichier de montage permanent — ajoute ces 3 lignes :
> #   UUID=xxx  /mnt/data   ext4  defaults  0  2   (montage auto sdb1 au démarrage)
> #   UUID=yyy  /mnt/perso  ext4  defaults  0  2   (montage auto sdb2 au démarrage)
> #   UUID=zzz  none        swap  sw        0  0   (activation auto swap au démarrage)
> 
> df -h && free -h && lsblk -f /dev/sdb
> # → df -h = vérifie l'espace monté | free -h = vérifie le swap actif | lsblk = structure complète
> ```

---

## Exercice 10.0b — Créer une architecture LVM

**Consigne :** Sur **SRVLX01**, crée une architecture LVM sur **sdc** : PV → VG **vg-data** → LV **lv-projets** (6 Go, ext4, monté sur `/srv/projets`, persistant).

Résultat attendu : `pvs` affiche sdc1, `vgs` affiche vg-data (~10 Go), `lvs` affiche lv-projets (6 Go), `df -hT /srv/projets` affiche le volume monté en ext4.

> [!tip]- Réponse
> ```bash
> apt install lvm2 -y
> # → Installe les outils LVM nécessaires : pvcreate, vgcreate, lvcreate, lvextend...
> 
> fdisk /dev/sdc
> # → Lance le partitionnement interactif sur sdc
> #   n → p → 1 → Entrée → Entrée  (partition primaire qui prend tout le disque)
> #   t → 8e                        (change le type en "Linux LVM" — obligatoire pour LVM)
> #   w                             (écrit la table de partitions et quitte)
> 
> pvcreate /dev/sdc1
> # → Initialise sdc1 comme Physical Volume (PV) — première brique LVM
> vgcreate vg-data /dev/sdc1
> # → Crée le Volume Group nommé "vg-data" en incluant sdc1 comme membre
> lvcreate -L 6G -n lv-projets vg-data
> # → Crée dans vg-data un Logical Volume nommé "lv-projets" de 6 Go
> mkfs.ext4 /dev/vg-data/lv-projets
> # → Formate le Logical Volume en système de fichiers ext4
> mkdir -p /srv/projets
> # → Crée le point de montage (dossier cible)
> mount /dev/vg-data/lv-projets /srv/projets
> # → Monte le Logical Volume sur /srv/projets — il devient accessible
> 
> nano /etc/fstab
> # → Ajoute la ligne suivante pour rendre le montage permanent au redémarrage :
> #   /dev/vg-data/lv-projets  /srv/projets  ext4  defaults  0  2
> 
> pvs && vgs && lvs
> # → pvs = liste des Physical Volumes | vgs = liste des Volume Groups | lvs = liste des Logical Volumes
> df -hT /srv/projets
> # → Vérifie la taille disponible et le type de système de fichiers sur /srv/projets
> ```
> ⚠️ Architecture toujours dans cet ordre : **PV → VG → LV → FS**

---

## Exercice 10.0c — Agrandir un volume LVM à chaud

**Consigne :** Sur **SRVLX01**, agrandis **lv-projets** de +4 Go en ajoutant le disque **sdd** au VG **vg-data**, sans démonter le volume.

> [!tip]- Réponse
> ```bash
> fdisk /dev/sdd
> # → Partitionne sdd pour créer un nouveau Physical Volume
> #   n → p → 1 → Entrée → Entrée  (partition primaire sur tout le disque)
> #   t → 8e                        (type Linux LVM — obligatoire)
> #   w                             (écrit et quitte)
> 
> pvcreate /dev/sdd1
> # → Initialise sdd1 comme nouveau Physical Volume LVM
> vgextend vg-data /dev/sdd1
> # → Ajoute sdd1 au Volume Group vg-data — lui apporte +10 Go d'espace disponible
> lvextend -L +4G /dev/vg-data/lv-projets
> # → Agrandit le Logical Volume lv-projets de +4 Go — sans démonter le volume
> resize2fs /dev/vg-data/lv-projets
> # → Étend le système de fichiers ext4 pour qu'il occupe le nouvel espace du LV
> 
> pvs && vgs && lvs
> # → Vérifie les nouvelles tailles : PV, VG et LV doivent tous afficher +4 Go
> df -hT /srv/projets
> # → Confirme que /srv/projets affiche bien la nouvelle taille
> ```
> ⚠️ `resize2fs` TOUJOURS APRÈS `lvextend` ! Sans ça, le LV est plus grand mais le FS ne le voit pas.

---

## Exercice 10.0d — Transformer le LVM en RAID 1

**Consigne :** Sur **SRVLX01**, supprime le LVM existant sur **sdc** et **sdd**, puis crée un **RAID 1** monté sur `/srv/backups` (persistant).

**Étape 1 — Détruire le LVM :**

> [!tip]- Réponse Étape 1
> ```bash
> umount /srv/projets
> # → Démonte le volume logique — obligatoire, on ne peut pas supprimer un LV monté
> nano /etc/fstab
> # → Supprime la ligne lv-projets — évite une erreur de montage au prochain redémarrage
> lvremove /dev/vg-data/lv-projets
> # → Supprime le Logical Volume lv-projets (libère l'espace dans le VG)
> vgremove vg-data
> # → Supprime le Volume Group vg-data (libère les PV associés)
> pvremove /dev/sdc1
> # → Retire sdc1 du rôle de Physical Volume LVM — la partition redevient ordinaire
> pvremove /dev/sdd1
> # → Retire sdd1 du rôle de Physical Volume LVM
> wipefs -a /dev/sdc
> # → Efface toutes les signatures LVM/partition sur sdc — repart d'un disque propre
> wipefs -a /dev/sdd
> # → Même chose sur sdd
> lsblk && pvs && vgs && lvs
> # → Vérifie que plus aucun PV, VG ni LV n'existe
> ```
> ⚠️ Toujours détruire dans l'ordre : **LV → VG → PV**

**Étape 2 — Créer le RAID 1 :**

> [!tip]- Réponse Étape 2
> ```bash
> apt install mdadm -y
> # → Installe l'outil de gestion des tableaux RAID logiciels Linux
> 
> fdisk /dev/sdc
> # → Partitionne sdc pour RAID
> #   n → p → 1 → Entrée → Entrée  (partition primaire sur tout le disque)
> #   t → fd                        (type "Linux RAID autodetect" — obligatoire pour mdadm)
> #   w                             (écrit et quitte)
> fdisk /dev/sdd
> # → Même chose sur sdd — les 2 membres du miroir doivent avoir le même type de partition
> 
> mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdc1 /dev/sdd1
> # → Crée le tableau RAID /dev/md0 : niveau 1 = miroir, avec 2 disques membres (sdc1 et sdd1)
> cat /proc/mdstat
> # → Affiche l'état en temps réel du RAID — attends que [UU] apparaisse (les 2 disques synchronisés)
> mkfs.ext4 /dev/md0
> # → Formate le tableau RAID entier en ext4 — on formate md0, pas sdc1 ou sdd1 directement
> mkdir -p /srv/backups
> # → Crée le point de montage
> mount /dev/md0 /srv/backups
> # → Monte le tableau RAID sur /srv/backups — il devient accessible comme dossier
> mdadm --detail --scan | tee -a /etc/mdadm/mdadm.conf
> # → Lit la config du RAID et l'ajoute dans mdadm.conf — nécessaire pour que le RAID survive au redémarrage
> update-initramfs -u
> # → Régénère l'image de démarrage pour que Linux reconnaisse le RAID dès le boot
> nano /etc/fstab
> # → Ajoute le montage permanent : /dev/md0  /srv/backups  ext4  defaults  0  2
> df -hT /srv/backups
> # → Vérifie que /srv/backups est bien monté en ext4 avec la bonne taille
> ```
> ⚠️ RAID 1 = **miroir** : capacité = 1 seul disque, tolère 1 panne

---

## Exercice 10.0e — Tout démonter et nettoyer

**Consigne :** Sur **SRVLX01**, nettoie entièrement les configurations de stockage des exercices précédents (RAID, LVM, partitions, swap) sur les disques sdb, sdc et sdd.

> [!tip]- Réponse
> ```bash
> umount /srv/backups
> # → Démonte le tableau RAID — obligatoire avant de l'arrêter
> mdadm --stop /dev/md0
> # → Arrête le tableau RAID md0 et le désactive en mémoire
> mdadm --zero-superblock /dev/sdc1
> # → Efface le superbloc RAID sur sdc1 — sans ça, Linux recrée le RAID au prochain démarrage
> mdadm --zero-superblock /dev/sdd1
> # → Même chose sur sdd1
> nano /etc/mdadm/mdadm.conf
> # → Supprime la ligne ARRAY md0 — évite la recréation automatique du RAID au boot
> update-initramfs -u
> # → Régénère l'initramfs pour prendre en compte la suppression du RAID
> umount /mnt/data && umount /mnt/perso && swapoff /dev/sdb3
> # → Démonte les 2 partitions ext4 de sdb et désactive l'espace swap sdb3
> nano /etc/fstab
> # → Supprime toutes les lignes ajoutées au cours des exercices (sdb1, sdb2, sdb3, md0)
> wipefs -a /dev/sdb && wipefs -a /dev/sdc && wipefs -a /dev/sdd
> # → Efface toutes les signatures FS/RAID/LVM sur les 3 disques — repart à zéro
> lsblk && cat /proc/mdstat
> # → lsblk = vérifie qu'aucune partition résiduelle n'existe | mdstat = vérifie qu'aucun RAID n'est actif
> ```

---

## Exercice 10.1 - Configuration réseau Linux

**Consigne :** Sur **SRVLX01**, configure une adresse IP statique selon le plan d'adressage.

Applique la configuration, redémarre le service réseau, puis vérifie la connectivité avec `ping -c 4 172.16.10.10`.

> [!tip]- Réponse
> ```bash
> apt install resolvconf -y
> nano /etc/network/interfaces
> ```
> ```
> auto enp0s3
> iface enp0s3 inet static
>     address 172.16.10.20
>     netmask 255.255.255.0
>     gateway 172.16.10.10
>     dns-nameservers 172.16.10.10
> ```
> ```bash
> systemctl restart networking
> ping -c 4 172.16.10.10
> ```

---

## Exercice 10.1b - Configurer resolv.conf et le fichier hosts

**Consigne :** Sur **SRVLX01**, configure la résolution DNS et la résolution locale par nom pour que les machines du réseau soient joignables par nom.

Résultat attendu : `ping SRVWIN01` et `ping srvwin01.sweetcakes.lan` répondent depuis SRVLX01 sans erreur DNS.

> [!tip]- Réponse
>
> **Étape 1 — resolv.conf**
> ```bash
> nano /etc/resolv.conf
> ```
> ```
> nameserver 172.16.10.10
> search sweetcakes.lan
> ```
> ⚠️ Sur Debian 12 avec `resolvconf` installé, ce fichier peut être écrasé au redémarrage.
> Pour le rendre persistant :
> ```bash
> nano /etc/resolvconf/resolv.conf.d/head
> # Ajouter :
> nameserver 172.16.10.10
> search sweetcakes.lan
> resolvconf -u
> ```
>
> **Étape 2 — /etc/hosts**
> ```bash
> nano /etc/hosts
> ```
> Ajouter à la fin :
> ```
> 172.16.10.10    SRVWIN01    srvwin01.sweetcakes.lan
> 172.16.10.30    UBUNTU-CLIENT
> 172.16.10.40    ipbx    ipbx.sweetcakes.lan
> ```
> ⚠️ Le fichier `/etc/hosts` est consulté **avant** le DNS — utile pour résoudre les noms même si le serveur AD est indisponible.
>
> **Étape 3 — Tester**
> ```bash
> ping -c 4 SRVWIN01              # → répond 172.16.10.10 ✅
> ping -c 4 srvwin01.sweetcakes.lan  # → répond 172.16.10.10 ✅
> cat /etc/resolv.conf            # → nameserver 172.16.10.10 ✅
> ```

---

## Exercice 10.2 - Créer un utilisateur Linux avec droits sudo

**Consigne :** Sur **SRVLX01**, crée l'utilisateur **technicien** avec les droits sudo.

> [!tip]- Réponse
> ```bash
> useradd -m -s /bin/bash technicien
> passwd technicien
> usermod -aG sudo technicien
> su - technicien
> sudo whoami  # → root
> ```

---

## Exercice 10.3 - Sécuriser SSH

> ⚠️ **À partir d'ici on travaille sur DEUX machines Linux :**
> - **SRVLX01** (172.16.10.20) → utilisateur **technicien**
> - **UBUNTU-CLIENT** (172.16.10.30) → utilisateur **wilder**
>
> Le but : comprendre le vrai échange de clés entre un client et un serveur distincts, tester la connexion par clé, puis envoyer un message chiffré via SCP.

---

### 10.3-PREP-A — Configurer l'IP statique sur UBUNTU-CLIENT

**Consigne :** Sur **UBUNTU-CLIENT**, configure une adresse IP statique selon le plan d'adressage.

Étapes :
1. Identifie le nom de ton interface réseau
2. Repère le fichier de configuration réseau actif
3. Applique la configuration IP statique
4. Corrige les permissions des fichiers de configuration réseau
5. Applique les changements

Résultat attendu : `ip a` → `ens33: inet 172.16.10.30/24` ✅

> [!tip]- Réponse
> ```bash
> ip a
> # → Repère le nom de ton interface (ex: ens33, enp0s3...)
>
> ls /etc/netplan/
> # → Tu verras : 01-network-manager-all.yaml  50-cloud-init.yaml
> # → C'est 50-cloud-init.yaml qu'il faut modifier
>
> sudo nano /etc/netplan/50-cloud-init.yaml
> ```
> Remplace tout le contenu par **(en remplaçant `<NOM_INTERFACE>` par le nom réel obtenu avec `ip a`)** :
> ```yaml
> network:
>   version: 2
>   renderer: NetworkManager
>   ethernets:
>     <NOM_INTERFACE>:
>       dhcp4: false
>       addresses:
>         - 172.16.10.30/24
>       routes:
>         - to: default
>           via: 172.16.10.10
>       nameservers:
>         addresses:
>           - 172.16.10.10
> ```
> ℹ️ Sous **VirtualBox** le nom est généralement `enp0s3`, sous **VMware** c'est `ens33`. Vérifie toujours avec `ip a` avant de modifier.
> Sauvegarde : `Ctrl+O` → `Entrée` → `Ctrl+X`
> ```bash
> sudo chmod 600 /etc/netplan/50-cloud-init.yaml
> sudo chmod 600 /etc/netplan/01-network-manager-all.yaml
> sudo netplan apply
> ip a
> # → ens33: inet 172.16.10.30/24 ✅
> ```
> ⚠️ **`gateway4` est déprécié** depuis Ubuntu 22.04 — utilise toujours `routes: - to: default via:`
> ⚠️ Le nom d'interface peut être différent (`ens33`, `enp0s3`...) → vérifie avec `ip a` avant
> ⚠️ YAML est sensible à l'indentation — 2 espaces par niveau, **jamais de tabulations**

---

### 10.3-PREP-B — Tester la connectivité

**Consigne :** Depuis **UBUNTU-CLIENT**, vérifie la connectivité réseau vers **SRVLX01** et **SRVWIN01**.

Résultat attendu : 0% packet loss sur les deux pings ✅

> [!tip]- Réponse
> ```bash
> ping -c 4 172.16.10.20   # → SRVLX01 ✅
> ping -c 4 172.16.10.10   # → SRVWIN01 ✅
> ```
> ⚠️ Si aucun ping ne répond → vérifier que la carte réseau VirtualBox est bien en mode **Réseau interne (intnet)**, même nom de réseau que les autres VMs.

---

### 10.3-PREP-C — Créer l'utilisateur wilder

**Consigne :** Sur **UBUNTU-CLIENT**, crée l'utilisateur **wilder**.

Résultat attendu : `id wilder` → affiche l'UID, le GID et les groupes de wilder ✅

> [!tip]- Réponse
> ```bash
> sudo adduser wilder
> # → Saisir le mot de passe wilder (ex: Azerty1*)
> # → Remplir ou laisser vide les infos (Full Name, Room...) → Entrée
> # → adduser crée automatiquement /home/wilder ✅
>
> id wilder
> # → uid=1001(wilder) gid=1001(wilder) groups=1001(wilder)
>
> sudo apt install openssh-server -y
> # → Installe le démon SSH sur UBUNTU-CLIENT (absent par défaut sur Ubuntu Desktop)
> sudo systemctl enable --now ssh
> # → Active et démarre le service SSH immédiatement
> ss -tlnp | grep :22
> # → LISTEN *:22 ✅ — UBUNTU-CLIENT accepte maintenant les connexions SSH entrantes
> ```
> ℹ️ `adduser` = commande interactive qui crée le home, demande le mot de passe et les infos automatiquement. C'est la commande recommandée sur Debian/Ubuntu.
> ⚠️ Ubuntu Desktop n'inclut **pas** `openssh-server` par défaut (seulement `openssh-client`). Sans cette étape, `ssh-copy-id wilder@172.16.10.30` depuis SRVLX01 retournera **"Connection refused"**.

---

### 10.3a — Générer les clés SSH sur SRVLX01

**Consigne :** Sur **SRVLX01**, en tant que **technicien**, génère une paire de clés SSH **ed25519**.

Résultat attendu : les deux fichiers sont présents dans `/home/technicien/.ssh/`.

> [!tip]- Réponse
> ```bash
> ssh-keygen -t ed25519
> # → Appuyer Entrée 3 fois (chemin par défaut, pas de passphrase)
> ls ~/.ssh/
> # → id_ed25519  id_ed25519.pub
> ```

---

### 10.3a-bis — Générer les clés SSH sur UBUNTU-CLIENT

**Consigne :** Sur **UBUNTU-CLIENT**, en tant que **wilder**, génère une paire de clés SSH **ed25519**.

Résultat attendu : les deux fichiers sont présents dans `/home/wilder/.ssh/`.

> [!tip]- Réponse
> ```bash
> ssh-keygen -t ed25519
> # → Entrée × 3
> ls ~/.ssh/
> # → id_ed25519  id_ed25519.pub
> ```

---

### 10.3b — Échange croisé des clés publiques

**Consigne :** Échange les clés publiques SSH dans les deux sens : wilder → technicien et technicien → wilder.

> [!tip]- Réponse
>
> **Sur UBUNTU-CLIENT (wilder) :**
> ```bash
> ssh-copy-id -p 22 technicien@172.16.10.20
> # → Mot de passe technicien demandé → "Number of key(s) added: 1" ✅
> ```
>
> **Sur SRVLX01 (technicien) :**
> ```bash
> ssh-copy-id -p 22 wilder@172.16.10.30
> # → Mot de passe wilder demandé → "Number of key(s) added: 1" ✅
> ```
>
> **Vérifier sur SRVLX01 :**
> ```bash
> cat /home/technicien/.ssh/authorized_keys
> # → La clé publique de wilder est présente
> ```
>
> **Vérifier sur UBUNTU-CLIENT :**
> ```bash
> cat /home/wilder/.ssh/authorized_keys
> # → La clé publique de technicien est présente
> ```

---

### 10.3c — Tester la connexion par clé dans les deux sens

**Consigne :** Vérifie que la connexion SSH par clé fonctionne sans mot de passe dans les deux sens.

Résultat attendu : aucun mot de passe demandé dans les deux cas ✅

> [!tip]- Réponse
>
> **Depuis UBUNTU-CLIENT :**
> ```bash
> ssh technicien@172.16.10.20   # → connexion directe sans MDP ✅
> exit
> ```
>
> **Depuis SRVLX01 :**
> ```bash
> ssh wilder@172.16.10.30   # → connexion directe sans MDP ✅
> exit
> ```

---

### 10.3c-bis — Envoyer un message chiffré via SCP

**Pourquoi ?** La connexion SSH chiffre tout le trafic — y compris les transferts de fichiers (SCP). Un fichier envoyé via SCP est **illisible sur le réseau** : seul le destinataire peut le lire.

**Consigne :** Depuis **UBUNTU-CLIENT** (wilder), envoie un fichier **message.txt** vers le répertoire home de **technicien** sur SRVLX01 via SCP.

Résultat attendu : `cat /home/technicien/message.txt` → affiche le message ✅

> [!tip]- Réponse
>
> **Sur UBUNTU-CLIENT (wilder) — créer et envoyer :**
> ```bash
> echo "Message confidentiel de wilder pour technicien" > ~/message.txt
> # → Crée le fichier avec le message
>
> scp ~/message.txt technicien@172.16.10.20:/home/technicien/message.txt
> # → Transfert sécurisé SSH — le fichier transite chiffré sur le réseau
> # → Aucun MDP demandé (clé SSH déjà en place) ✅
> ```
>
> **Sur SRVLX01 (technicien) — vérifier la réception :**
> ```bash
> ls ~/message.txt
> # → Le fichier est présent
> cat ~/message.txt
> # → "Message confidentiel de wilder pour technicien" ✅
> ```
>
> **Ce que tu comprends ici :**
> - Le transfert SCP passe par le tunnel SSH → le contenu est **chiffré en transit**
> - Sans la clé privée de wilder, personne ne peut intercepter le fichier sur le réseau
> - C'est différent d'un simple `cp` ou d'un partage réseau non sécurisé (SMB sans chiffrement)

---

### 10.3d — Configurer sshd_config

**Consigne :** Sur **SRVLX01**, sécurise SSH : port **22504**, root interdit, auth par mot de passe désactivée, auth par clé uniquement, accès restreint à **technicien**.

Redémarre le service SSH et vérifie que le port **22504** est bien en écoute.

> [!tip]- Réponse
> ```bash
> nano /etc/ssh/sshd_config
> ```
> ```
> Port 22504
> PermitRootLogin no
> PasswordAuthentication no
> PubkeyAuthentication yes
> AllowUsers technicien
> ```
> ```bash
> systemctl restart ssh
> ss -tlnp | grep ssh  # → LISTEN *:22504
> ```

### 10.3e — Tester la sécurisation

**Consigne :** Depuis **UBUNTU-CLIENT**, teste les 3 scénarios de connexion SSH sur le port **22504** pour valider la configuration sécurisée.

> [!tip]- Réponse
> ```bash
> # Depuis UBUNTU-CLIENT (wilder) — la clé de wilder est dans authorized_keys de technicien ✅
> ssh -p 22504 technicien@172.16.10.20                                         # → ✅ sans MDP
> ssh -p 22504 root@172.16.10.20                                                # → ✅ Permission denied (PermitRootLogin no)
> ssh -p 22504 -o PubkeyAuthentication=no -o PasswordAuthentication=yes technicien@172.16.10.20   # → ✅ Permission denied (PasswordAuthentication no)
> ```
> ⚠️ Les tests s'exécutent depuis **UBUNTU-CLIENT** et non depuis SRVLX01 lui-même.
> Raison technique : après 10.3b, `authorized_keys` de technicien contient la **clé de wilder** (pas la sienne propre). C'est donc wilder qui peut se connecter en tant que technicien sans mot de passe.

---

## Exercice 10.4 - Gestion des droits sur un fichier

**Consigne :** Sur **SRVLX01**, crée l'utilisateur **webmaster**, crée le fichier `/home/partage/Lisez-moi`, attribue-lui la propriété **webmaster** et les permissions **660**.

> [!tip]- Réponse
> ```bash
> useradd -m -s /bin/bash webmaster
> passwd webmaster
> mkdir -p /home/partage
> touch /home/partage/Lisez-moi
> chown webmaster:webmaster /home/partage/Lisez-moi
> chmod 660 /home/partage/Lisez-moi
> ls -l /home/partage/Lisez-moi
> ```

## Exercice 10.4b - Ajouter un 2ème utilisateur

**Consigne :** Sur **SRVLX01**, crée l'utilisateur **stagiaire**.

> [!tip]- Réponse
> ```bash
> adduser stagiaire
> id stagiaire && ls /home/stagiaire
> ```

## Exercice 10.4c - Renommer un utilisateur

**Consigne :** Sur **SRVLX01**, renomme le compte **stagiaire** en **assistant** (login, home, groupe primaire).

> [!tip]- Réponse
> ```bash
> usermod -l assistant stagiaire
> usermod -d /home/assistant -m assistant
> groupmod -n assistant stagiaire
> id assistant
> ```

## Exercice 10.4d - Créer un groupe et gérer les membres

**Consigne :** Sur **SRVLX01**, crée le groupe **equipe-web**, ajoute **webmaster** et **assistant**, puis retire **assistant** du groupe.

> [!tip]- Réponse
> ```bash
> groupadd equipe-web
> usermod -aG equipe-web webmaster
> usermod -aG equipe-web assistant
> getent group equipe-web
> gpasswd -d assistant equipe-web
> getent group equipe-web
> ```

---

## Exercice 10.5 - Remplacer un site web par un site temporaire

> [!warning] ⚠️ PRÉREQUIS — Fichiers fournis par le formateur
> Avant de commencer cet exercice, les deux fichiers suivants doivent être présents dans `/root` sur **SRVLX01** :
> - `/root/LisezMoi` — fichier texte d'instructions
> - `/root/En-Travaux.tar.gz` — archive du site "En Travaux"
>
> Ces fichiers sont fournis par le formateur. Méthode de transfert suggérée :
> ```bash
> # Depuis UBUNTU-CLIENT ou CLIENT01, via SCP (port 22504, user technicien) :
> scp -P 22504 LisezMoi technicien@172.16.10.20:/tmp/
> scp -P 22504 En-Travaux.tar.gz technicien@172.16.10.20:/tmp/
> # Puis sur SRVLX01 (en root), déplacer vers /root :
> mv /tmp/LisezMoi /root/
> mv /tmp/En-Travaux.tar.gz /root/
> ```
> ⚠️ Utiliser le **port 22504** (configuré en 10.3d) et le compte **technicien** (PermitRootLogin no).
> Si les fichiers sont absents → demander au formateur avant de commencer.

### 10.5a — Lire et sauvegarder

**Consigne :** Sur **SRVLX01**, installe **Apache2** et sauvegarde le site par défaut avant de le remplacer.

Résultat attendu : une copie de sauvegarde du site Apache par défaut est créée.

> [!tip]- Réponse
> ```bash
> cat /root/LisezMoi
> apt install apache2 -y
> # ⚠️ Installe apache2 d'abord : crée /var/www/html nécessaire pour la sauvegarde
> cp -r /var/www/html /var/www/html_backup
> ```

### 10.5b — Extraire et copier

**Consigne :** Sur **SRVLX01**, déploie le contenu de l'archive **En-Travaux.tar.gz** comme nouveau site web par défaut d'Apache.

Résultat attendu : les fichiers du site "En Travaux" sont en place dans le répertoire web.

> [!tip]- Réponse
> ```bash
> cd /root
> tar -xzf En-Travaux.tar.gz
> cp -r En-Travaux/* /var/www/html/
> ```

### 10.5c — Permissions et test

**Consigne :** Sur **SRVLX01**, applique les bonnes permissions et propriétaire sur le répertoire web, puis démarre Apache2.

> [!tip]- Réponse
> ```bash
> chown -R www-data:www-data /var/www/html/
> find /var/www/html/ -type d -exec chmod 755 {} \;
> find /var/www/html/ -type f -exec chmod 644 {} \;
> systemctl enable --now apache2
> ```
> Tester : http://172.16.10.20

---

# PARTIE 11 : TÉLÉPHONIE IPBX

### 📋 Utilisateurs et extensions VoIP

| Utilisateur AD | Nom | Extension | Secret | Machine |
|----------------|-----|-----------|--------|---------|
| **p.durand** | Pierre DURAND | **80104** | P@ssw0rd80104 | CLIENT01 |
| **l.moreau** | Lucie MOREAU | **80103** | P@ssw0rd80103 | CLIENT02 |

> ⚠️ MicroSIP remplacé par **3CX** comme softphone dans ce lab.

---

## Exercice 11.0 — Préparer la VM FreePBX (réseau + firewall + port SIP)

**Consigne :** Sur la VM **IPBX**, prépare l'environnement : IP statique, trafic SIP autorisé, port PJSIP 5060 vérifié.

Résultat attendu : la VM IPBX est joignable en ping depuis CLIENT01 et FreePBX écoute sur le port 5060.

> [!tip]- Réponse
>
> **Étape 1 — IP statique**
> ```bash
> ip a
> # → Repère le nom de l'interface (ex: eth0, ens33...)
> nano /etc/network/interfaces
> ```
> ```
> auto <NOM_INTERFACE>
> iface <NOM_INTERFACE> inet static
>     address 172.16.10.40
>     netmask 255.255.255.0
>     gateway 172.16.10.10
>     dns-nameservers 172.16.10.10
> ```
> ```bash
> systemctl restart networking
> ip a   # → 172.16.10.40/24 doit apparaître ✅
> ping -c 4 172.16.10.10   # → SRVWIN01 répond ✅
> ```
>
> **Étape 2 — Désactiver iptables (firewall)**
> ```bash
> iptables -F
> iptables -X
> iptables -P INPUT ACCEPT
> iptables -P FORWARD ACCEPT
> iptables -P OUTPUT ACCEPT
> # Rendre persistant au redémarrage :
> apt install iptables-persistent -y
> netfilter-persistent save
> ```
> ⚠️ Sans cette étape, le trafic SIP (port 5060 UDP) est bloqué → les softphones obtiennent "Not connected".
>
> **Étape 3 — Vérifier le port PJSIP dans FreePBX**
> - Depuis CLIENT01 → http://172.16.10.40
> - **Settings → Advanced Settings** → SIP Channel Driver : **chan_pjsip** ✅
> - **Settings → Asterisk SIP Settings → onglet PJSIP** → Port d'écoute : **5060** ✅
> - Si modifié → cliquer **Submit** puis **Apply Config**
>
> Vérifier en ligne de commande sur l'IPBX :
> ```bash
> ss -ulnp | grep 5060
> # → UNCONN 0 0 0.0.0.0:5060 → Asterisk écoute ✅
> ```

---

## Exercice 11.1 - Créer les comptes utilisateurs sur l'IPBX

**Consigne :** Sur **FreePBX**, crée les extensions VoIP pour **p.durand** et **l.moreau**.

Résultat attendu : les deux extensions apparaissent dans la liste Extensions de FreePBX avec le statut actif.

> [!tip]- Réponse
> - http://172.16.10.40 → Applications → Extensions → Add Extension → **New Chan_PJSIP Extension**
> - Créer 80104 (Pierre DURAND) → Submit → **Apply Config**
> - Créer 80103 (Lucie MOREAU) → Submit → **Apply Config**
> - ⚠️ **Apply Config** obligatoire après chaque extension
> - ⚠️ Choisir **New Chan_PJSIP Extension** — FreePBX 17 n'utilise plus Chan_SIP legacy

---

## Exercice 11.2 - Configurer les softphones et tester un appel

### 11.2a — Configurer les deux softphones

**Consigne :** Sur **CLIENT01** et **CLIENT02**, configure **3CX** avec l'extension de chaque utilisateur et vérifie l'enregistrement.

> [!tip]- Réponse
> **CLIENT01 (p.durand) :**
> - 3CX → Add Account
> - Account name : p.durand | Extension : 80104 | ID : 80104 | Password : P@ssw0rd80104
> - I am in the office - local IP : 172.16.10.40
> - Laisser tout le reste vide — ne pas cocher "Use 3CX Tunnel" ni "Outbound Proxy"
> - OK → **Registered** ✅
>
> **CLIENT02 (l.moreau) :**
> - 3CX → Add Account
> - Account name : l.moreau | Extension : 80103 | ID : 80103 | Password : P@ssw0rd80103
> - I am in the office - local IP : 172.16.10.40
> - Laisser tout le reste vide — ne pas cocher "Use 3CX Tunnel" ni "Outbound Proxy"
> - OK → **Registered** ✅

### 11.2b — Passer un appel test

**Consigne :** Depuis **CLIENT01** (p.durand), passe un appel vers **l.moreau** sur CLIENT02 et vérifie que la communication s'établit.

> [!tip]- Réponse
> - CLIENT01 (p.durand / 80104) compose **80103** → CLIENT02 (l.moreau) décroche ✅
> - Capture d'écran des deux 3CX en communication

---

## Exercice 11.3 - Configurer un renvoi d'appel (Call Forward)

### 11.3a — Configurer dans FreePBX

**Consigne :** Configure un **renvoi d'appel inconditionnel** sur l'extension de **p.durand** vers **l.moreau**.

Résultat attendu : tout appel vers 80104 est automatiquement renvoyé vers 80103.

> [!tip]- Réponse
> - http://172.16.10.40 → Applications → Extensions → **80104** → tab **Advanced**
> - Call Forward All → Enabled → Destination : **80103** → Submit → **Apply Config**
>
> | Code | Action |
> |------|--------|
> | `*72` + numéro | Renvoi inconditionnel |
> | `*73` | Désactiver renvoi |
> | `*90` + numéro | Renvoi si occupé |
> | `*92` + numéro | Renvoi si pas de réponse |
>
> ⚠️ Vérifier `*90` / `*92` dans : **Admin → Feature Codes** avant utilisation.

### 11.3b — Tester le renvoi d'appel

**Consigne :** Vérifie le renvoi d'appel, puis désactive-le.

Résultat attendu : un appel vers 80104 ne sonne pas sur CLIENT01 — renvoi automatique vers 80103.

> [!tip]- Réponse
> - CLIENT02 (l.moreau / 80103) appelle **80104** → CLIENT01 (p.durand) **ne sonne pas** → renvoi ✅
> - ⚠️ Boucle infinie possible entre 80103↔80104 : utiliser un 3ème softphone si disponible
> - Désactiver : FreePBX → 80104 → tab Advanced → Call Forward All : **Disabled** → Apply Config