
> Uniquement ce qui tombe à l'examen. Issu des Checkpoints 1→4.
> Réponses cachées → entraîne-toi à répondre AVANT d'ouvrir.

---

## 🔴 SUPPORT / ITIL

---

### Q.1 — Différence incident vs problème (ITIL) ?

> [!success]- Réponse
> - **Incident** : perturbation non planifiée d'un service → effet visible, à traiter vite
> - **Problème** : cause inconnue d'un ou plusieurs incidents → peut être latent (pas encore déclenché d'incident)
> - "Un incident est l'effet. Un problème est la cause."
> - La gestion des problèmes est axée sur la **prévention**. La gestion des incidents sur la **restauration rapide**.

---

### Q.2 — Étapes de résolution d'incident par téléphone ?

> [!success]- Réponse
> 1. Identification et enregistrement de l'incident
> 2. Catégorisation et priorisation
> 3. Recueil d'informations (questions à l'utilisateur)
> 4. Diagnostic et tentative de résolution
> 5. Escalade si nécessaire
> 6. Résolution et vérification avec l'utilisateur
> 7. Clôture et documentation

---

### Q.3 — Quels sont les moyens de prise en main à distance ?

> [!success]- Réponse
> - **Windows** : RDP (port TCP 3389) → `mstsc`
> - **Linux** : SSH (port TCP 22) → `ssh user@ip`
> - Outils tiers : TeamViewer, AnyDesk, VNC
> - Depuis l'Active Directory : outils d'administration RSAT

---

## 🔴 ACTIVE DIRECTORY

---

### Q.4 — Qu'est-ce qu'un rôle FSMO ? Citez les 5.

> [!success]- Réponse
> FSMO = Flexible Single Master Operation. Rôles spéciaux sur les DC AD.
>
> **À l'échelle de la forêt (1 seul par forêt) :**
> - Maître de schéma → gère les MAJ du schéma AD
> - Maître de nommage de domaine → gère l'ajout/suppression de domaines
>
> **À l'échelle du domaine (1 seul par domaine) :**
> - Maître RID → attribue les pools RID/SID aux DC
> - Maître d'infrastructure → gère les relations objets entre domaines
> - Émulateur PDC → synchronisation du temps, verrouillages de comptes
>
> ⚠️ Par défaut le 1er DC cumule les 5. Bonne pratique : les répartir.

---

### Q.5 — Pourquoi la réplication entre DC est-elle primordiale ?

> [!success]- Réponse
> Si le DC unique est éteint ou corrompu → le domaine est **inutilisable** (authentification impossible).
> La réplication assure la **continuité de service** et la **cohérence des données** (comptes, GPO, mots de passe) sur tous les DC du domaine.
> Bonne pratique : minimum 2 DC par domaine.

---

### Q.6 — Qu'est-ce qu'une GPO ?

> [!success]- Réponse
> Ensemble de paramètres appliqués aux utilisateurs et aux ordinateurs d'un domaine AD.
> - **Config ordinateur** : appliquée au démarrage de la machine
> - **Config utilisateur** : appliquée à l'ouverture de session
>
> Exemples : fond d'écran, lecteurs réseau, restrictions, déploiement logiciels, scripts.
> Appliquées par ordre LSDOU : Local → Site → Domaine → OU

---

### Q.7 — Différence utilisateur AD vs utilisateur local ?

> [!success]- Réponse
> - **Local** : stocké dans la base SAM locale, valable sur 1 seul PC, pas de GPO domaine
> - **AD** : stocké dans NTDS.dit sur le DC, valable sur tout le domaine, soumis aux GPO

---

## 🔴 RÉSEAU

---

### Q.8 — Qu'est-ce qu'un VLAN ? Pourquoi l'utiliser ?

> [!success]- Réponse
> Réseau local virtuel = segmentation logique d'un réseau physique.
> - **Sécurité** : isolation du trafic (VLAN 10 RH ne voit pas VLAN 20 IT)
> - **Performance** : réduit les domaines de broadcast
> - **Flexibilité** : regroupement logique indépendant du câblage physique
>
> Communication inter-VLAN → routeur (Router-on-a-Stick) ou switch L3.

---

### Q.9 — Qu'est-ce qu'un trunk ? Quel protocole ?

> [!success]- Réponse
> Lien réseau transportant **plusieurs VLANs** entre 2 équipements (switch↔switch ou switch↔routeur).
> Protocole : **802.1Q** → ajoute un tag de 4 octets dans la trame Ethernet avec le numéro de VLAN.
> - Port **access** = 1 VLAN, vers un PC → pas de tag
> - Port **trunk** = plusieurs VLANs, entre équipements → tag 802.1Q

---

### Q.10 — Comment faire communiquer 2 VLANs ?

> [!success]- Réponse
> Les VLANs sont isolés par défaut. Deux solutions :
> - **Router-on-a-Stick** : routeur avec sous-interfaces (G0/0.10, G0/0.20) + trunk vers switch. `encapsulation dot1Q [VLAN]` sur chaque sous-interface. Chaque sous-interface = passerelle du VLAN.
> - **Switch L3** : `ip routing` + SVIs (`interface vlan 10` avec IP = passerelle)

---

### Q.11 — Qu'est-ce qu'une ACL Cisco ? Standard vs étendue ?

> [!success]- Réponse
> Filtre le trafic sur un routeur selon des règles permit/deny.
> - **Standard (1-99)** : filtre sur IP **source** uniquement → placer près de la **destination**
> - **Étendue (100-199)** : filtre sur source + destination + protocole + port → placer près de la **source**
> - ⚠️ `deny any` implicite à la fin → sans `permit ip any any`, **tout est bloqué**

---

### Q.12 — Qu'est-ce que le NAT/PAT ?

> [!success]- Réponse
> - **NAT** : traduit une IP privée en IP publique pour sortir sur Internet
> - **PAT** (NAT overload) : plusieurs machines internes partagent **une seule IP publique** grâce aux ports
> - **DNAT / Port forwarding** : redirige le trafic entrant vers un serveur interne
> - Commande Cisco PAT : `ip nat inside source list 1 interface G0/1 overload`

---

### Q.13 — Qu'est-ce qu'une route statique ? Différence avec route par défaut ?

> [!success]- Réponse
> Route ajoutée **manuellement** dans la table de routage.
> Syntaxe Cisco : `ip route [réseau] [masque] [next-hop]`
> - **C** = Connected, **S** = Static dans `show ip route`
> - **Route par défaut** : `ip route 0.0.0.0 0.0.0.0 [next-hop]` → tout le trafic inconnu part vers ce routeur
> - ⚠️ Routes dans les 2 sens obligatoires (aller ET retour)

---

### Q.14 — Wildcard mask : c'est quoi ? Comment calculer ?

> [!success]- Réponse
> Inverse du masque de sous-réseau. Utilisé dans les ACL et OSPF Cisco.
> Calcul : **255.255.255.255 − masque**
>
> | Masque | Wildcard | Signification |
> |--------|----------|---------------|
> | 255.255.255.0 (/24) | 0.0.0.255 | Tout un réseau /24 |
> | 255.255.255.255 (/32) | 0.0.0.0 | 1 seule IP (`host`) |
> | 255.255.0.0 (/16) | 0.0.255.255 | Tout un /16 |
>
> ⚠️ `172.17.0.0 0.255.255.255` ne couvre PAS uniquement 172.17.x.x → couvre tout 172.x.x.x (premier octet fixé, les 3 autres libres)

---

### Q.15 — 2 PCs sur réseaux différents ne se pinguent pas. Que vérifier ?

> [!success]- Réponse
> 1. **Câblage** : voyants verts ?
> 2. **IP/masque** : `ipconfig` / `ip a` → bonne IP, bon masque ?
> 3. **Passerelle** : configurée ? Ping la passerelle → si KO, problème local
> 4. **Routeur** : interfaces UP ? (`show ip interface brief`) Routes vers les réseaux distants ?
> 5. **Retour** : le routeur distant a-t-il une route retour vers le réseau source ?
> ⚠️ Piège classique : on configure l'aller, on oublie le retour.

---

### Q.16 — Où placer une capture Wireshark ? Comment localiser la position ?

> [!success]- Réponse
> Les **adresses IP** restent identiques de bout en bout.
> Les **adresses MAC changent à chaque routeur**.
>
> | MAC destination vue | Position de la capture |
> |---------------------|----------------------|
> | MAC du routeur | Entre le PC source et son routeur |
> | MAC d'un autre routeur | Entre deux routeurs |
> | MAC du PC destination | Après le dernier routeur |
>
> Ethertype à connaître : `0x0800` = IPv4, `0x0806` = ARP, `0x86DD` = IPv6

---

### Q.17 — Qu'est-ce qu'un VPN ? Types ?

> [!success]- Réponse
> Connexion sécurisée et chiffrée entre deux points à travers Internet (tunnel).
> - **Site-à-site** : relie deux réseaux (siège ↔ filiale) → toujours actif
> - **Nomade (client-à-site)** : utilisateur distant → réseau entreprise → à la demande
> - Protocoles : IPSec, OpenVPN, WireGuard, SSL/TLS

---

### Q.18 — À quoi sert le DHCP ? Ports ?

> [!success]- Réponse
> Attribue automatiquement une configuration IP : adresse, masque, passerelle, DNS, durée de bail.
> Ports : **UDP 67** (serveur) / **UDP 68** (client).
> Processus : DORA (Discover → Offer → Request → Acknowledge)

---

### Q.19 — Qu'est-ce qu'un proxy ? À quoi ça sert ?

> [!success]- Réponse
> Intermédiaire entre les utilisateurs internes et Internet.
> Fonctions : cache (performance), filtrage d'URL (sécurité), anonymat, journalisation.
> Port courant : 3128 (Squid), 8080.

---

### Q.20 — Différence switch vs hub ?

> [!success]- Réponse
> - **Hub** : couche 1, répète le signal sur tous les ports, collisions possibles, obsolète
> - **Switch** : couche 2, table MAC, envoi ciblé uniquement vers le bon port, bande passante dédiée

---

### Q.21 — C'est quoi le câble console Cisco ?

> [!success]- Réponse
> Câble console (rollover) pour configuration initiale d'un équipement Cisco sans réseau.
> RJ-45 côté switch/routeur → DB9 ou USB côté PC.
> Logiciel : PuTTY en mode Serial. Paramètres : **9600 bauds, 8 bits, No parity, 1 stop bit (8N1)**.

---

## 🔴 SÉCURITÉ

---

### Q.22 — À quoi sert un pare-feu ? Types ?

> [!success]- Réponse
> Filtre le trafic entrant/sortant selon des règles : bloque le non-autorisé, autorise le légitime, journalise.
> - **Matériel** (appliance) : Cisco ASA, Palo Alto, pfSense
> - **Logiciel** : Windows Defender Firewall, iptables/nftables (Linux)
> - **Stateful** : suit l'état des connexions → plus intelligent qu'un simple filtre de paquets
> - **Applicatif (DPI)** : inspecte jusqu'à la couche 7, WAF pour HTTP

---

### Q.23 — SSH : comment le sécuriser ? Ordre de configuration Cisco ?

> [!success]- Réponse
> SSH = administration chiffrée (vs Telnet en clair).
> Ordre obligatoire sur Cisco :
> 1. `hostname` (obligatoire pour la clé RSA)
> 2. `ip domain-name lab.lan`
> 3. `username admin privilege 15 secret MDP`
> 4. `crypto key generate rsa modulus 2048` (max 2048 sur Packet Tracer)
> 5. `ip ssh version 2`
> 6. `line vty 0 4` → `transport input ssh` + `login local` + `exec-timeout 5 0`

---

## 🔴 VIRTUALISATION / SERVEURS

---

### Q.24 — Avantages d'un cluster d'hyperviseurs ?

> [!success]- Réponse
> - **HA** (Haute disponibilité) : si un hôte tombe, les VMs migrent automatiquement
> - **Répartition de charge** : équilibrage des ressources entre hôtes
> - **Migration à chaud** (vMotion/Live Migration) : déplacer une VM sans coupure
> - **Tolérance aux pannes** : redondance matérielle
> Ex : VMware vSphere, Proxmox VE, Hyper-V Cluster

---

### Q.25 — Conteneur vs Machine Virtuelle ?

> [!success]- Réponse
> | | VM | Conteneur |
> |--|----|-----------| 
> | Isolation | Kernel complet dédié | Partage le kernel de l'hôte |
> | Taille | Plusieurs Go | Quelques Mo |
> | Démarrage | Minutes | Secondes |
> | Usage | OS complet isolé | Application isolée |
>
> Les conteneurs sont plus légers mais moins isolés que les VMs.

---

### Q.26 — Docker : image, conteneur, volume ?

> [!success]- Réponse
> - **Image** : template figé (système de fichiers + config). Ex : `nginx:latest`. Construite via un `Dockerfile`.
> - **Conteneur** : instance en cours d'exécution d'une image. Le conteneur est à l'image ce que le processus est au programme.
> - **Volume** : stockage persistant géré par Docker. Survit à la destruction du conteneur.
>
> Commandes clés : `docker run`, `docker ps`, `docker images`, `docker pull`, `docker build`

---

## 🔴 SAUVEGARDE

---

### Q.27 — Types de sauvegardes : complète, incrémentale, différentielle ?

> [!success]- Réponse
> - **Complète** : tout dupliquer. Long, consommateur. Restauration simple.
> - **Incrémentale** : uniquement les modifications depuis la **sauvegarde précédente** (complète ou incrémentale). Rapide. Restauration délicate (besoin de toute la chaîne).
> - **Différentielle** : uniquement les modifications depuis la dernière **complète**. Compromis.
>
> Classique en entreprise : **1 complète/semaine + 1 incrémentale/jour**

---

### Q.28 — Qu'est-ce que le RTO et le RPO ?

> [!success]- Réponse
> - **RTO** (Recovery Time Objective) : temps maximum acceptable avant que le service soit rétabli. Ex : RTO = 4h → service doit être restauré en moins de 4h.
> - **RPO** (Recovery Point Objective) : perte de données acceptable. Ex : RPO = 1h → on accepte de perdre au maximum 1h de données.
> Ces indicateurs définissent le PRA (Plan de Reprise d'Activité) et PCA (Plan de Continuité d'Activité).

---

## 🔴 DÉPLOIEMENT

---

### Q.29 — Différence WDS, MDT, SCCM ?

> [!success]- Réponse
> | Outil | Rôle | Remarque |
> |-------|------|----------|
> | **WDS** | Déploiement réseau via PXE (images WIM) | Natif Windows Server, simple |
> | **MDT** | Séquences de tâches automatisées (LiteTouch) | Gratuit, nécessite WADK |
> | **SCCM** | Gestion complète du parc + déploiement (ZeroTouch) | Payant, grands parcs |
>
> MDT seul = pas de PXE → combiner MDT + WDS pour le boot réseau.

---

### Q.30 — Comment fonctionne le boot PXE ?

> [!success]- Réponse
> PXE (Preboot Execution Environment) = démarrage d'un PC via le réseau.
> 3 étapes :
> 1. Le PC demande une IP et un fichier d'amorçage via **DHCP/BOOTP**
> 2. Téléchargement du fichier de boot via **TFTP**
> 3. Exécution du fichier (WinPE ou autre) → installation de l'OS

---

### Q.31 — Qu'est-ce que WSUS ?

> [!success]- Réponse
> Rôle Windows Server pour gérer les mises à jour Microsoft de façon **centralisée**.
> - Téléchargement unique depuis Microsoft → redistribution interne
> - Contrôle et approbation des MAJ avant déploiement
> - Planification des installations, rapports de conformité
> - Alternative : SCCM pour les grands parcs

---

## 🔴 SUPERVISION / JOURNALISATION

---

### Q.32 — À quoi sert la supervision réseau ? Outils ?

> [!success]- Réponse
> Surveiller en temps réel l'état des équipements et services de l'infrastructure.
> - **Disponibilité** : est-ce que le service répond ?
> - **Performance** : CPU, RAM, bande passante, temps de réponse
> - **Alertes** : notification en cas de problème
>
> Protocole : **SNMP** (Simple Network Management Protocol) → collecte d'infos sur les équipements réseau
> Outils : Nagios, Zabbix, PRTG, SolarWinds

---

### Q.33 — À quoi servent les journaux (logs) ? Où les trouver ?

> [!success]- Réponse
> Enregistrement des événements système et réseau pour diagnostic, audit, sécurité.
> - **Windows** : Observateur d'événements (`eventvwr`) → Application, Sécurité, Système
> - **Linux** : `/var/log/` → `syslog`, `auth.log`, `kern.log`
> - Commandes Linux : `journalctl`, `tail -f /var/log/syslog`
> - Outil centralisé : SIEM, Graylog, ELK Stack

---

## 🔴 LINUX

---

### Q.34 — Qu'est-ce que chmod ? Expliquer les droits Linux ?

> [!success]- Réponse
> `chmod` modifie les permissions d'un fichier/dossier sous Linux.
> - **r** = lecture (4), **w** = écriture (2), **x** = exécution (1)
> - 3 niveaux : **u** (user), **g** (group), **o** (others)
> - `chmod u+x fichier.sh` → ajoute le droit d'exécution au propriétaire
> - `chmod 755` = rwxr-xr-x (propriétaire tout, groupe+autres : lecture+exécution)

---

### Q.35 — Diagnostic réseau Linux : commandes ?

> [!success]- Réponse
> | Commande | Usage |
> |----------|-------|
> | `ip a` | Afficher les interfaces et IPs |
> | `ip r` | Table de routage |
> | `ping [IP]` | Test connectivité ICMP |
> | `ss -tuln` | Ports en écoute |
> | `dig` / `nslookup` | Résolution DNS |
> | `traceroute` | Chemin des paquets |
> | `ip addr add 172.16.8.16/24 dev enp0s8` | Ajouter une IP temporaire |

---

### Q.36 — Accès refusé à un dossier Linux : comment diagnostiquer ?

> [!success]- Réponse
> `ls: impossible d'ouvrir le répertoire : Permission non accordée`
>
> 1. `ls -la /home/wilder/` → voir les permissions du dossier
> 2. `stat /home/wilder/travaux` → propriétaire, groupe, permissions
> 3. `id wilder` → groupes de l'utilisateur
> 4. Correction : `chmod o+rx /home/wilder/travaux` ou `chown wilder:wilder travaux`
> ⚠️ Un dossier nécessite **x** (exécution) pour être traversé, pas seulement **r**.

---

## 🔴 RÉSEAU — COMPLÉMENTS

---

### Q.37 — Qu'est-ce qu'un serveur DNS ? Comment fonctionne-t-il ?

> [!success]- Réponse
> Traduit un nom de domaine en adresse IP (résolution de noms).
> - **Résolution récursive** : le client demande au DNS local → qui interroge les autres si nécessaire
> - Enregistrements clés : **A** (IPv4), **AAAA** (IPv6), **CNAME** (alias), **MX** (mail), **PTR** (résolution inverse)
> - Port : **UDP/TCP 53**
> - `ping IP OK mais ping nom KO` → problème DNS, pas réseau

---

### Q.38 — Sous-réseaux : comment découper un réseau ?

> [!success]- Réponse
> Pour découper `192.168.1.0/24` en 4 sous-réseaux :
> - 4 sous-réseaux = 2² → emprunter 2 bits → nouveau masque = /26 (255.255.255.192)
> - Chaque sous-réseau = 64 adresses dont 62 utilisables
>
> | Sous-réseau | Plage | Broadcast |
> |-------------|-------|-----------|
> | .0/26 | .1 → .62 | .63 |
> | .64/26 | .65 → .126 | .127 |
> | .128/26 | .129 → .190 | .191 |
> | .192/26 | .193 → .254 | .255 |

---

### Q.39 — Qu'est-ce que la messagerie électronique ? Protocoles ?

> [!success]- Réponse
> | Protocole | Rôle | Port |
> |-----------|------|------|
> | **SMTP** | Envoi d'emails | TCP 25 (587 sécurisé) |
> | **IMAP** | Réception, emails sur serveur | TCP 143 (993 SSL) |
> | **POP3** | Réception, emails téléchargés | TCP 110 (995 SSL) |
>
> IMAP = emails restent sur le serveur (multi-appareil).
> POP3 = emails téléchargés et supprimés du serveur.

---

### Q.40 — Différence HTTP vs HTTPS ? TLS ?

> [!success]- Réponse
> - **HTTP** : port 80, flux en clair, interceptable
> - **HTTPS** : port 443, chiffré par **TLS** (Transport Layer Security)
> - TLS utilise des certificats (X.509) pour authentifier le serveur et chiffrer la communication
> - En entreprise : le serveur web doit avoir un certificat signé par une CA (autorité de certification)
> ⚠️ HTTP → tout le monde peut lire les mots de passe. HTTPS → chiffré dès la poignée de main (handshake).