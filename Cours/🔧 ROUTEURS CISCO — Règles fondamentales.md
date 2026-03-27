

## 🎯 L'essentiel

1. **Chaque interface = 1 réseau différent** — impossible d'avoir 2 interfaces dans le même réseau sur un même routeur
2. **Lien entre 2 routeurs = même réseau** — les 2 interfaces du lien doivent être dans le même sous-réseau (réseau de transit)
3. **1 seul routeur = pas de routes statiques** — les réseaux directement connectés sont connus automatiquement
4. **2 routeurs ou plus = routes statiques obligatoires** — chaque routeur doit apprendre les réseaux qu'il ne connaît pas
5. **Routes dans les 2 sens** — configurer l'aller ET le retour sur chaque routeur
6. **`no shutdown` obligatoire** — les interfaces sont DOWN par défaut sur un routeur Cisco

---

## 🔧 Commandes à retenir

| Commande | Rôle |
|----------|------|
| `interface GigabitEthernet 0/0` | Entrer dans une interface |
| `ip address [IP] [masque]` | Assigner une IP |
| `no shutdown` | Activer l'interface |
| `do wr` | Sauvegarder depuis config mode |
| `end` | Revenir au mode enable |
| `show ip interface brief` | Voir toutes les interfaces + statut |
| `show ip route` | Voir la table de routage |
| `ip route [réseau] [masque] [next-hop]` | Ajouter une route statique |
| `no ip address` | Supprimer une IP d'une interface |

---

## 🔧 Schéma type — 2 routeurs

```
PC1─────SW1────G0/0[R1]G0/1────G0/0[R2]G0/1────SW2─────PC2

Réseau PC1  : 192.168.1.0/24   → R1 G0/0 = 192.168.1.254
Réseau transit : 10.0.0.0/24   → R1 G0/1 = 10.0.0.1 / R2 G0/0 = 10.0.0.2
Réseau PC2  : 192.168.2.0/24   → R2 G0/1 = 192.168.2.254
```

**Routes statiques à ajouter :**
```
Sur R1 : ip route 192.168.2.0 255.255.255.0 10.0.0.2
Sur R2 : ip route 192.168.1.0 255.255.255.0 10.0.0.1
```

---

## 🔧 Config complète type — R1

```
R1(config)# interface GigabitEthernet 0/0
R1(config-if)# ip address 192.168.1.254 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit

R1(config)# interface GigabitEthernet 0/1
R1(config-if)# ip address 10.0.0.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit

R1(config)# ip route 192.168.2.0 255.255.255.0 10.0.0.2
R1(config)# end
R1# wr
```

---

## ⚠️ Piège classique

> **Ne jamais mettre le next-hop = une IP de ce même routeur**
> → Erreur : `%Invalid next hop address (it's this router)`
> Le next-hop doit toujours être l'IP du **routeur voisin** sur le lien de transit.

> **Ne pas oublier les routes retour**
> Si R1 sait aller chez R2 mais R2 ne sait pas revenir → le ping échoue dans les 2 sens.

> **`up / down` sur une interface**
> Status = up (câble branché) mais Protocol = down → l'autre côté n'est pas configuré.
> Dès que les 2 interfaces du lien ont une IP + `no shutdown` → passe en `up / up`.

> **`wr` depuis le mode interface = erreur**
> Utiliser `do wr` depuis config-if, ou `exit` puis `wr` depuis enable.

---

## 📝 QUIZ Checkpoint

### Question 1
**Avec 1 seul routeur, faut-il des routes statiques ?**

> [!success]- 🔓 Réponse
> **Non.** Avec 1 seul routeur, tous les réseaux connectés sont directement connus (ligne C dans `show ip route`). Il suffit de configurer les interfaces et les passerelles sur les PCs.

---

### Question 2
**Avec 2 routeurs, pourquoi faut-il des routes statiques ?**

> [!success]- 🔓 Réponse
> R1 ne connaît que ses réseaux directement connectés. Il ne connaît pas le réseau derrière R2. Il faut lui ajouter manuellement : `ip route [réseau distant] [masque] [IP de R2 sur le lien de transit]`.

---

### Question 3
**Qu'est-ce qu'un réseau de transit ?**

> [!success]- 🔓 Réponse
> Réseau utilisé uniquement pour relier 2 routeurs entre eux. Les PCs ne sont pas dessus. Il contient exactement 2 IPs : une pour chaque routeur.
> Ex : `10.0.0.0/24` → R1 = `10.0.0.1`, R2 = `10.0.0.2`

---

### Question 4
**Une interface est en `up/down`. Que faire ?**

> [!success]- 🔓 Réponse
> L'autre côté du lien n'est pas configuré. Il faut configurer l'interface du routeur voisin avec une IP dans le même réseau + `no shutdown`. Les 2 côtés doivent être actifs pour que le protocole monte.

---

### Question 5
**Erreur : `%Invalid next hop address (it's this router)`. Pourquoi ?**

> [!success]- 🔓 Réponse
> Le next-hop renseigné dans la route statique est une IP qui appartient à ce même routeur. Le next-hop doit être l'IP de l'interface du routeur **voisin** sur le lien de transit.

---

### Question 6
**2 PCs sur des réseaux différents ne se pinguent pas via 2 routeurs. Quoi vérifier ?**

> [!success]- 🔓 Réponse
> 1. Interfaces des 2 routeurs UP/UP (`show ip interface brief`)
> 2. Lien de transit : les 2 IPs dans le même réseau
> 3. Route statique sur R1 vers le réseau de PC2 (via R2)
> 4. Route statique sur R2 vers le réseau de PC1 (via R1)
> 5. Passerelle correcte sur chaque PC
> ⚠️ Vérifier les 2 sens : aller ET retour

---

### Question 7
**Dans `show ip route`, que signifient les lettres C, S, L ?**

> [!success]- 🔓 Réponse
> - **C** = Connected → réseau directement connecté à une interface
> - **S** = Static → route ajoutée manuellement avec `ip route`
> - **L** = Local → IP locale de l'interface elle-même (/32)

---

# 🔀 VLANs & TRUNK

## 🎯 L'essentiel

1. **VLAN** = segmentation logique → isolation du trafic L2, réduction des broadcasts
2. **Port access** = 1 seul VLAN → vers un PC
3. **Port trunk** = plusieurs VLANs → entre switch↔switch ou switch↔routeur (protocole 802.1Q)
4. **Sans routeur = pas de communication inter-VLAN** — les VLANs sont totalement isolés
5. **VLAN 1 = VLAN natif** → ne jamais y mettre d'IP → faille VLAN hopping

## 🔧 Commandes switch

| Commande | Rôle |
|----------|------|
| `vlan 10` + `name MARKETING` | Créer et nommer un VLAN |
| `switchport mode access` + `switchport access vlan 10` | Port vers un PC |
| `switchport mode trunk` | Port multi-VLAN |
| `show vlan brief` | Vérifier VLANs et ports |
| `show interfaces trunk` | Vérifier les trunks actifs |
| `do wr` | Sauvegarder depuis config mode |

## ⚠️ Piège classique

> Sur un **switch 3560** (L3) : `switchport trunk encapsulation dot1q` est **obligatoire** avant `switchport mode trunk`. Le 2960 n'en a pas besoin (dot1Q uniquement).

> Un port trunk sort automatiquement du VLAN access — normal, c'est voulu.

---

# 🔀 ROUTER-ON-A-STICK (inter-VLAN)

## 🎯 L'essentiel

1. **1 seul câble trunk** entre le switch et le routeur
2. **1 sous-interface par VLAN** sur le routeur : `G0/0.10`, `G0/0.20`
3. **`encapsulation dot1Q [VLAN]`** obligatoire sur chaque sous-interface
4. **`no shutdown` sur l'interface physique** (G0/0) — obligatoire sinon rien ne monte
5. **Chaque sous-interface = passerelle du VLAN**

## 🔧 Config Router-on-a-Stick

```
! Interface physique : active SANS IP
R1(config)# interface GigabitEthernet 0/0
R1(config-if)# no shutdown
R1(config-if)# exit

! Sous-interface VLAN 10
R1(config)# interface GigabitEthernet 0/0.10
R1(config-subif)# encapsulation dot1Q 10
R1(config-subif)# ip address 192.168.10.254 255.255.255.0
R1(config-subif)# exit

! Sous-interface VLAN 20
R1(config)# interface GigabitEthernet 0/0.20
R1(config-subif)# encapsulation dot1Q 20
R1(config-subif)# ip address 192.168.20.254 255.255.255.0
R1(config-subif)# exit
R1(config)# end
R1# wr
```

## ⚠️ Piège classique

> Oublier `no shutdown` sur **G0/0 physique** → aucune sous-interface ne monte, même avec `no shutdown` sur les sous-interfaces.

> Le port du switch côté routeur doit être en **trunk**, pas en access.

---

# 🔀 SWITCH L3 (inter-VLAN sans routeur)

## 🎯 L'essentiel

1. Switch L3 (modèle 3560) = fait le routing ET le switching
2. `ip routing` **obligatoire** pour activer le routage
3. **SVI** (Switched Virtual Interface) = interface logique par VLAN avec une IP = passerelle
4. Plus performant que Router-on-a-Stick car routage dans le matériel

## 🔧 Config Switch L3

```
SW-CORE(config)# ip routing

SW-CORE(config)# interface vlan 10
SW-CORE(config-if)# ip address 192.168.10.254 255.255.255.0
SW-CORE(config-if)# no shutdown
SW-CORE(config-if)# exit

SW-CORE(config)# interface vlan 20
SW-CORE(config-if)# ip address 192.168.20.254 255.255.255.0
SW-CORE(config-if)# no shutdown
SW-CORE(config-if)# exit
SW-CORE(config)# end
SW-CORE# wr
```

## ⚠️ Piège classique

> Oublier `ip routing` → le switch voit les VLANs mais ne route pas entre eux.

---

# 🔒 VLAN ADMIN + SSH

## 🎯 L'essentiel

1. **VLAN 99** = VLAN d'administration, isolé du trafic utilisateurs
2. **VLAN 666** = DEAD_ZONE → ports inutilisés mis dedans + `shutdown`
3. **SVI VLAN 99** = IP d'administration du switch (cible SSH)
4. **VLAN 1** = toujours désactiver (`no ip address` + `shutdown`)
5. **SSH** = administration chiffrée. Ordre obligatoire : hostname → domain-name → username → crypto key → ssh version → line vty

## 🔧 Config SSH sur switch

```
SW1(config)# ip domain-name lab.lan
SW1(config)# username admin privilege 15 secret Azerty1*
SW1(config)# crypto key generate rsa modulus 2048
SW1(config)# ip ssh version 2
SW1(config)# line vty 0 4
SW1(config-line)# login local
SW1(config-line)# transport input ssh
SW1(config-line)# exec-timeout 5 0
SW1(config-line)# exit
SW1(config)# do wr
```

## ⚠️ Piège classique

> `transport input none` → bloque TOUT y compris SSH. Utiliser `transport input ssh`.

> `crypto key generate rsa modulus 4096` → erreur dans Packet Tracer. Maximum **2048**.

> Sans `ip domain-name` → impossible de générer les clés RSA.

---

# 🔒 ACL (Access Control List)

## 🎯 L'essentiel

1. **Standard (1-99)** : filtre sur IP **source** uniquement → placer près de la **destination**
2. **Étendue (100-199)** : filtre sur source + destination + protocole + port → placer près de la **source**
3. **`deny any` implicite** à la fin → sans `permit ip any any`, tout est bloqué
4. **Wildcard mask** = inverse du masque → `255.255.255.255 − masque`
5. Les règles sont lues **dans l'ordre** → la 1ère qui matche s'applique

## 🔧 Commandes ACL

```
! ACL étendue : bloquer ICMP de PC3 vers VLAN 10
R1(config)# access-list 100 deny icmp host 192.168.20.1 192.168.10.0 0.0.0.255
R1(config)# access-list 100 permit ip any any

! Appliquer sur l'interface (près de la source)
R1(config)# interface GigabitEthernet 0/0.20
R1(config-subif)# ip access-group 100 in
R1(config-subif)# exit
R1(config)# end
R1# wr

! Supprimer
R1(config)# interface GigabitEthernet 0/0.20
R1(config-subif)# no ip access-group 100 in
R1(config-subif)# exit
R1(config)# no access-list 100

! Vérifier
R1# show access-lists
```

## ⚠️ Piège classique

> `172.17.0.0 0.255.255.255` → ne couvre PAS uniquement 172.17.x.x — couvre **tout 172.x.x.x** (premier octet fixé, les 3 autres libres).

> ACL standard près de la source → bloque trop large car elle filtre uniquement sur la source.

---

# 🌐 NAT / PAT

## 🎯 L'essentiel

1. **PAT** (overload) = plusieurs IP privées partagent **1 seule IP publique** via les ports
2. **Port forwarding** (NAT statique) = redirige le trafic entrant vers un serveur interne
3. Marquer les interfaces : `ip nat inside` (LAN) et `ip nat outside` (WAN)
4. L'ACL identifie le trafic à NATer
5. `overload` = mot-clé obligatoire pour le PAT

## 🔧 Config NAT/PAT

```
! Interfaces
R1(config)# interface GigabitEthernet 0/0.10
R1(config-subif)# ip nat inside
R1(config-subif)# exit

R1(config)# interface GigabitEthernet 0/1
R1(config-if)# ip nat outside
R1(config-if)# exit

! ACL + PAT
R1(config)# access-list 1 permit 192.168.10.0 0.0.0.255
R1(config)# access-list 1 permit 192.168.20.0 0.0.0.255
R1(config)# ip nat inside source list 1 interface GigabitEthernet 0/1 overload

! Port forwarding entrant
R1(config)# ip nat inside source static tcp 192.168.10.200 80 8.8.8.1 80
R1(config)# end
R1# wr

! Vérifier
R1# show ip nat translations
```

## ⚠️ Piège classique

> `ip nat inside` doit être sur les **sous-interfaces** (G0/0.10, G0/0.20) dans Packet Tracer — pas sur l'interface physique G0/0.

> Sans ACL correspondante, le PAT ne translate rien.