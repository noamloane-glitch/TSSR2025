# VPN - Virtual Private Network

## Document de révision TSSR - Titre RNCP

---

**Formation** : Technicien Supérieur Systèmes et Réseaux (TSSR)

**Sujet** : VPN - Les réseaux privés virtuels

**Date** : Janvier 2026

**Type** : Synthèse de cours complète

---

## 📋 Sommaire

1. Le concept de VPN
2. IPsec
3. TLS  
4. OpenVPN
5. Aller plus loin
6. Points clés à retenir
7. Glossaire technique

---

## Le concept de VPN

> [!quote] Définition
> Un **réseau privé virtuel (VPN)** consiste à simuler un réseau privé en se basant sur des réseaux réels quelconques et donc éventuellement publics comme Internet.

### Objectif d'un VPN

**Triple objectif** :
- **Transparent** : l'utilisateur ne voit pas la complexité
- **Même niveau de sécurité** qu'un réseau local
- **Malgré la traversée de réseaux publics**

**En résumé** : Faire comme si on était sur le réseau local

### La notion de tunnel

Un **tunnel** est un passage sûr à travers un réseau.

**Principe du tunneling** :
1. Considérer le PDU à transmettre comme une donnée
2. Chiffrement et authentification
3. Transmission dans un autre PDU

**VPN de différents niveaux** :
- Trame => VPN de niveau 2
- Paquet => VPN de niveau 3

---

## Les types de VPN

### VPN site à site

**Interconnexion de réseaux physiques**
- Les extrémités sont des passerelles
- Connexion permanente
- Transparent pour les utilisateurs

**Exemple** : Entreprise à Paris avec filiale à Montréal
- Connexion de 2 groupes d'ordinateurs
- Permanent et transparent
- Travail sur même intranet

### VPN accès distant

**Une machine accède à un réseau**
- Connexion à la demande
- Utilisable de n'importe où
- Protection même depuis WiFi non sécurisé

**Exemple** : Employé en télétravail
- Connexion au réseau via tunnel chiffré
- L'utilisateur contrôle la connexion

### VPN point à point

**Communication uniquement entre deux machines**

**Exemple** : Administrateur se connectant à un serveur précis
- Sans exposer le reste du réseau

---

## Les protocoles VPN

**VPN sécurisés** :
- **IPsec** : fonctionnalités dans IP
- **OpenVPN** : basé sur TLS
- **WireGuard** : moderne et performant

**Tunnels seuls** (non chiffrés) :
- L2TP, GRE
- MPLS (VPN opérateur)
- SSH (ponctuel)

---

## IPsec

### Définition

**Internet Protocol Security** - Standards IETF de 1995

**Constitué de 3 protocoles** :
1. **IKEv2** : Négociation d'une communication IPsec
2. **AH** : Authentification et contrôle d'intégrité
3. **ESP** : Confidentialité + Authentification + Intégrité

### IKEv2 - Internet Key Exchange

**Rôle** : Négociation de l'établissement d'une connexion

**Processus** :
- Mise en place d'un canal confidentiel (Diffie-Hellman)
- Négociation des paramètres cryptographiques
- Authentification mutuelle via PSK ou certificats

**Communication** : UDP port 500 (et 4500 pour NAT traversal)

### AH - Authentication Header

**RFC 4302** - Protocole 51

**Mécanisme** : HMAC pour authentification et intégrité

**ICV calculé sur** :
- Le contenu (PDU)
- L'entête AH
- L'entête IP (partie fixe)

**Protection contre le rejeu** : numéro de séquence

**Limitation** : ❌ Pas de chiffrement

### ESP - Encapsulating Security Payload

**RFC 4303** - Protocole 50

**Fonctionnement** :
- PDU chiffré avec algorithme symétrique
- ICV calculé sur contenu chiffré + entête ESP
- Protection contre le rejeu

**Protections** :
- ✅ Confidentialité
- ✅ Authentification
- ✅ Intégrité
- ✅ Anti-rejeu

**ESP est recommandé** (inclut toutes les protections d'AH + chiffrement)

### Les modes IPsec

**Mode Transport** :
- Protocole de niveau 4 dans AH/ESP puis IP
- Entêtes IP en clair
- Communication hôte à hôte

**Mode Tunnel** :
- Paquet IP complet encapsulé dans AH/ESP puis dans IP
- Ensemble authentifié, intègre et chiffré
- Communication passerelle à passerelle

**Mode tunnel = Standard pour VPN site-à-site**

### Security Association (SA)

Une **SA** regroupe :
- Cryptosystèmes et paramètres utilisés
- Clés nécessaires
- Utilisation d'AH et/ou ESP
- Mode tunnel ou transport

**SPI** (Security Parameter Index) : identifiant de la SA

---

## TLS

### Définition

**Transport Layer Security** - Successeur de SSL

**Historique** :
- SSL développé par Netscape
- TLS repris par IETF en 1999

**Caractéristiques** :
- Protocole de transport (couche 5-6 OSI)
- Au-dessus de TCP ou UDP (DTLS)

**Protections** :
- Authentification par certificats
- Contrôle d'intégrité
- Protection contre le rejeu
- Confidentialité persistante (PFS)

### Les versions

**Obsolètes** (à éviter) :
- ❌ SSL 1.0, SSL 2.0, SSL 3.0
- ❌ TLS 1.0, TLS 1.1

**Utilisables** :
- ⚠️ TLS 1.2 (compatibilité)

**Recommandées** :
- ✅ **TLS 1.3** (standard actuel)
- ✅ DTLS 1.3

### Négociation TLS (Handshake)

**Processus** :
1. Établissement connexion confidentielle (ECDH)
2. Négociation version TLS et cryptosystèmes
3. Authentification du serveur (certificat X.509)
4. Authentification du client (optionnelle)

### Communication TLS

**Après établissement** :
- Échanges confidentiels via algo symétrique (AES)
- Intégrité et authenticité via HMAC

### Utilisations de TLS

- **HTTPS** : HTTP over TLS (TCP 443)
- **FTPS** : FTP over TLS (TCP 989, 990)
- **SMTPS** : SMTP over TLS (TCP 465, 587)
- **POP3S/IMAPS** : TCP 995, 993
- **DoT** : DNS over TLS (TCP/UDP 853)
- **OpenVPN** : VPN basé sur TLS

---

## OpenVPN

### Définition

**Logiciel VPN libre** client/serveur

**Caractéristiques** :
- Basé sur TLS
- Publié en 2001
- Multiplateforme
- UDP ou TCP, port 1194
- Tunnel niveau 2 (pont) ou 3 (routeur)

### Authentification

**Méthodes** :
- Login/mot de passe
- PSK (Pre-Shared Key)
- **Certificats X.509** (recommandé)

**Configuration classique** : authentification par certificats (PKI)

### Infrastructure PKI

**Mise en place** :
1. Création certificat et clé privée de l'AC
2. Pour chaque serveur/client :
   - Création CSR
   - Certification par l'AC
   - Déploiement certificat + certificat AC

**Easy-RSA** : outil facilitant la gestion PKI

### Révocation

**Méthodes** :
- **Manuel** : CRL (Certificate Revocation List)
- **Automatique** : OCSP

### Mode routeur

**Interfaces TUN** :
- Pseudo-interfaces tun sur client et serveur
- Comportement comme 2 ports d'un routeur
- Paquets IP transmis à travers le tunnel

**Usage** : Standard (le plus courant)

### Mode pont

**Interfaces TAP** :
- Pseudo-interfaces tap sur client et serveur
- Comportement comme 2 ports d'un switch
- Trames Ethernet transmises à travers le tunnel

**Usage** : Cas spécifiques (broadcast nécessaire)

---

## Aller plus loin

### Serveur bastion

**Principe** : Brique de sécurité, point d'accès unique

**Objectifs** :
- Réduire la surface d'attaque
- Centraliser les accès
- Améliorer la traçabilité

**Exemples** :
- Apache Guacamole
- Azure Bastion

### Jump server

**Définition** : Serveur intermédiaire

**Principe** : On s'y connecte avant d'accéder aux serveurs internes

**Exemples** :
- Serveur SSH
- Serveur RDP

**Remarque** :
- Tous les jump servers sont des bastions
- Tous les bastions ne sont pas de simples jump servers

---

## Points clés à retenir

### Concept de VPN

- VPN = réseau privé virtuel sur réseau public
- Tunnel = passage sûr avec encapsulation + chiffrement
- 3 types : site-à-site, accès distant, point-à-point

### IPsec

**Composants** :
- IKEv2 : négociation (UDP 500/4500)
- AH : auth + intégrité (proto 51)
- ESP : auth + intégrité + confidentialité (proto 50)

**Modes** :
- Transport : hôte à hôte (rare)
- Tunnel : site à site (standard)

### TLS

**Versions** :
- ❌ SSL, TLS 1.0/1.1 (obsolètes)
- ✅ TLS 1.3 (recommandé)

**Usage** : HTTPS, OpenVPN, etc.

### OpenVPN

- Basé sur TLS
- Authentification par certificats (PKI)
- Mode TUN (routeur) : standard
- Mode TAP (pont) : cas spécifiques

### Sécurité

- Bastion : point d'accès unique
- Jump server : serveur intermédiaire
- 2FA recommandé
- Surveillance et logs essentiels

---

## Glossaire technique

| Terme | Définition |
|-------|------------|
| **VPN** | Virtual Private Network - Réseau privé virtuel |
| **Tunnel** | Passage sécurisé à travers un réseau |
| **IPsec** | Internet Protocol Security - Suite de protocoles niveau 3 |
| **IKEv2** | Internet Key Exchange - Négociation IPsec |
| **AH** | Authentication Header - Auth et intégrité (proto 51) |
| **ESP** | Encapsulating Security Payload - Auth + intégrité + confidentialité (proto 50) |
| **SA** | Security Association - Paramètres de sécurité |
| **SPI** | Security Parameter Index - Identifiant SA |
| **TLS** | Transport Layer Security - Protocole couche 5-6 |
| **SSL** | Secure Sockets Layer - Ancien nom de TLS |
| **PSK** | Pre-Shared Key - Clé partagée |
| **PKI** | Public Key Infrastructure - Infrastructure à clés publiques |
| **CA** | Certificate Authority - Autorité de certification |
| **CSR** | Certificate Signing Request - Demande de signature |
| **CRL** | Certificate Revocation List - Liste de révocation |
| **TUN** | Interface niveau 3 (IP) |
| **TAP** | Interface niveau 2 (Ethernet) |
| **Bastion** | Point d'accès unique sécurisé |
| **Jump Server** | Serveur intermédiaire de rebond |

---

## Commandes utiles

```bash
# OpenVPN
sudo openvpn --config client.ovpn

# Easy-RSA
./easyrsa init-pki
./easyrsa build-ca
./easyrsa gen-req client1
./easyrsa sign-req client client1
./easyrsa revoke client1
./easyrsa gen-crl

# Vérifier connectivité
ip addr show tun0
ping 10.8.0.1

# Logs
sudo journalctl -u openvpn@server -f
```

---

## Conclusion

**Points essentiels** :
- VPN = sécurisation des communications sur Internet
- IPsec = standard entreprise (site-à-site)
- OpenVPN = flexible multiplateforme (accès distant)
- TLS 1.3 = version recommandée
- Certificats > PSK
- Bastion = sécurité renforcée

**Attention** : Un VPN ouvre une brèche entre réseaux
- Authentification robuste obligatoire
- Surveillance et logs essentiels
- Cloisonnement réseau recommandé

---

> [!success] Prêt pour le titre RNCP !
> Tu maîtrises maintenant les concepts de VPN, IPsec, TLS et OpenVPN. Pratique sur ton homelab pour consolider ! 🔐🚀

---

**📚 Document créé pour la préparation au titre RNCP TSSR**

**🎯 Objectif** : Maîtriser les VPN pour l'examen et la pratique professionnelle

**✅ Compatible Obsidian** avec callouts natifs et liens internes
