# Procédures de configuration Switch (Cisco IOS)

> Labs réalisés sous **Packet Tracer** — Switches **Catalyst 2960**  
> Progression : réseau à plat → VLANs → sécurité administration → accès SSH

---

## 1. Navigation et commandes de base

### Modes IOS

```
Switch> enable                  # Passer en mode privilégié
Switch# configure terminal      # Passer en mode configuration globale
Switch(config)# exit            # Revenir au menu précédent
Switch(config-if)# end          # Revenir directement au mode privilégié
Switch# logout                  # Déconnexion de la session
```

### Abréviations courantes

| Commande complète | Abréviation |
|---|---|
| `configure terminal` | `conf t` |
| `interface gigabitEthernet 0/1` | `int g0/1` |
| `interface fastEthernet 0/1` | `int f0/1` |
| `show` | `sh` |

**Astuce :** La touche `TAB` complète automatiquement une commande.  
**Aide :** `?` seul liste les commandes disponibles, `show ?` liste les options de `show`, `conf?` affiche la suite d'une commande.

### Mot-clé `do`

En mode configuration (`config` ou `config-if`), préfixer toute commande `show` avec `do` :

```
SW1(config)# do show vlan brief
SW1(config)# do show ip interface brief
SW1(config)# do wr
```

---

## 2. Configuration globale

### Nommer le switch

```
SW1(config)# hostname SW1
```

### Sauvegarder la configuration

```
SW1# copy running-config startup-config
# ou simplement :
SW1# write
```

### Supprimer la configuration de démarrage

```
SW1# write erase
SW1# reload
```

### Redémarrage

```
SW1# reload                   # Immédiat
SW1# reload in 5              # Dans 5 minutes
SW1# reload cancel            # Annulation
SW1# show reload              # Statut
```

---

## 3. Configuration des interfaces (ports)

### Sélectionner une interface

```
SW1(config)# interface fastEthernet 0/1        # Un port unique
SW1(config)# interface range fastEthernet 0/1 - 12   # Plage consécutive
SW1(config)# interface range f0/1-6, f0/19-22  # Plages non-consécutives
```

### Activation / désactivation d'un port

```
SW1(config-if)# no shutdown    # Activer (up)
SW1(config-if)# shutdown       # Désactiver (down)
```

### Vérification des interfaces

```
SW1# show ip interface brief          # Résumé : IP, statut up/down
SW1# show interfaces status           # Ports, VLAN, duplex, vitesse
SW1# show interfaces fastEthernet 0/1 # Détail complet d'un port
SW1# show running-config interface g0/1
```

---

## 4. Configuration des VLANs

### Créer et nommer des VLANs

```
SW1(config)# vlan 10
SW1(config-vlan)# name MARKETING
SW1(config-vlan)# exit

SW1(config)# vlan 20
SW1(config-vlan)# name RH
SW1(config-vlan)# exit
```

Plusieurs VLANs en une ligne : `vlan 10,20,99`

Supprimer un VLAN : `no vlan 10`

### Vérification des VLANs

```
SW1# show vlan
SW1# show vlan brief
```

---

## 5. Configuration des ports en mode Access

Chaque port access n'appartient qu'à **un seul VLAN**. Le tag est géré de façon transparente pour le PC.

```
SW1(config)# interface fastEthernet 0/1
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# exit

# En masse avec interface range :
SW1(config)# interface range fastEthernet 0/1 - 12
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport access vlan 10
SW1(config-if-range)# exit

SW1(config)# interface range fastEthernet 0/13 - 24
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport access vlan 20
SW1(config-if-range)# exit
```

---

## 6. Configuration des ports en mode Trunk

Un port trunk transporte **plusieurs VLANs** sur un seul lien physique (802.1Q). Utilisé entre switches, ou entre switch et routeur.

```
SW1(config)# interface gigabitEthernet 0/1
SW1(config-if)# switchport mode trunk
SW1(config-if)# exit
SW1(config)# do wr
```

> Sur certains matériels, ajouter avant la commande trunk :
> `SW1(config-if)# switchport trunk encapsulation dot1q`

### Filtrage des VLANs autorisés sur un trunk

```
SW1(config-if)# switchport trunk allowed vlan all       # Tout autoriser
SW1(config-if)# switchport trunk allowed vlan none      # Tout bloquer
SW1(config-if)# switchport trunk allowed vlan 10        # Autoriser VLAN 10 uniquement
SW1(config-if)# switchport trunk allowed vlan add 20    # Ajouter VLAN 20
SW1(config-if)# switchport trunk allowed vlan remove 10 # Retirer VLAN 10
```

### Vérification du trunk

```
SW1# show interfaces trunk
```

Sortie attendue :

```
Port     Mode    Encapsulation  Status      Native vlan
Gig0/1   on      802.1q         trunking    1

Port     Vlans allowed on trunk
Gig0/1   1-1005

Port     Vlans allowed and active in management domain
Gig0/1   1,10,20
```

---

## 7. VLAN d'administration (SVI)

### Pourquoi un VLAN dédié à l'administration ?

Le VLAN 1 (VLAN natif par défaut) **ne doit jamais** porter l'adresse d'administration :
- Il est accessible depuis n'importe quel port non configuré.
- Il est vulnérable à l'attaque **VLAN hopping** (double tagging 802.1Q).

→ Créer un VLAN dédié (ex. VLAN 99) pour l'accès SSH/Telnet au switch.

### Créer l'interface SVI (Switched Virtual Interface)

Une SVI est une interface **logicielle** associée à un VLAN (pas à un port physique). C'est l'adresse IP qui sera la cible des connexions d'administration.

```
SW1(config)# vlan 99
SW1(config-vlan)# name ADMIN
SW1(config-vlan)# exit

SW1(config)# interface vlan 99
SW1(config-if)# ip address 192.168.99.254 255.255.255.0
SW1(config-if)# no shutdown
SW1(config-if)# exit
```

Sortie attendue :
```
%LINK-5-CHANGED: Interface Vlan99, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan99, changed state to up
```

### Désactiver le VLAN 1

```
SW1(config)# interface vlan 1
SW1(config-if)# no ip address
SW1(config-if)# shutdown
SW1(config-if)# exit
```

Vérification (aucune ligne `Internet address` ne doit apparaître) :
```
SW1# show interfaces vlan 1
→ Vlan1 is administratively down, line protocol is down
```

### Connecter le poste d'administration

Assigner le port physique de PC-ADMIN au VLAN 99 :

```
SW1(config)# interface fastEthernet 0/24
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 99
SW1(config-if)# exit
```

---

## 8. DEAD_ZONE — Isolation des ports inutilisés

### Principe

Tout port non utilisé doit être :
1. Assigné à un VLAN de confinement sans ressource IP (ex. VLAN 666 ou 999)
2. Désactivé avec `shutdown`

> **Pourquoi ?** Un port `shutdown` sans VLAN défini reste techniquement dans le VLAN 1. En l'assignant d'abord au VLAN DEAD_ZONE, même si quelqu'un fait un `no shutdown`, le port atterrit dans un VLAN vide sans aucune ressource accessible.  
> → **Defense in depth**

### Configuration

```
# Étape 1 : Créer le VLAN DEAD_ZONE
SW1(config)# vlan 666
SW1(config-vlan)# name DEAD_ZONE
SW1(config-vlan)# exit

# Étape 2 : Assigner les ports inutilisés et les éteindre
SW1(config)# interface range fastEthernet 0/7 - 12
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport access vlan 666
SW1(config-if-range)# shutdown
SW1(config-if-range)# exit

SW1(config)# interface range fastEthernet 0/19 - 23
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport access vlan 666
SW1(config-if-range)# shutdown
SW1(config-if-range)# exit

SW1(config)# end
SW1# write memory
```

### Vérification

```
SW1# show vlan brief
→ Les ports DEAD_ZONE apparaissent sous VLAN 666

SW1# show interfaces status
→ Les ports DEAD_ZONE ont le statut "disabled"

SW1# show ip interface brief
→ Les ports DEAD_ZONE sont "administratively down"
```

> **Note :** Les ports trunk ne doivent **pas** être mis dans le VLAN 666.  
> En entreprise on trouve aussi **999** ou **4094** pour ce rôle.

---

## 9. Configuration SSH

### Prérequis et ordre obligatoire

La configuration SSH doit impérativement être réalisée dans cet ordre :

1. Hostname défini
2. Nom de domaine IP
3. Génération des clés RSA
4. Création utilisateur local
5. Activation SSH v2 + lignes VTY

### Configuration complète SW1

```
# 1. Nom de domaine (obligatoire pour la génération RSA)
SW1(config)# ip domain-name lab.lan

# 2. Utilisateur local avec privilege 15 (accès direct au mode enable)
SW1(config)# username admin privilege 15 secret Azerty1*

# 3. Génération de la paire de clés RSA
SW1(config)# crypto key generate rsa
→ How many bits in the modulus [512]: 4096
# Ou directement :
SW1(config)# crypto key generate rsa modulus 4096

# 4. Activation SSH v2
SW1(config)# ip ssh version 2

# 5. Configuration des lignes VTY
SW1(config)# line vty 0 4
SW1(config-line)# login local           # Auth via base locale
SW1(config-line)# transport input ssh   # Interdit Telnet, SSH uniquement
SW1(config-line)# exec-timeout 5 0     # Déconnexion après 5 min d'inactivité
SW1(config-line)# exit

SW1(config)# do wr
```

> **`secret` vs `password` :** `secret` chiffre le mot de passe en MD5 dans la config. `password` le stocke en clair → toujours utiliser `secret`.  
> **`privilege 15` :** Accorde directement le mode enable sans double authentification. En production, préférer `privilege 1` avec un `enable secret` séparé pour une meilleure traçabilité.  
> **`transport input ssh` ≠ `transport input none` :** Le premier autorise SSH, le second bloque tout accès distant.

### Vérifications SSH

```
SW1# show ip ssh
→ SSH Enabled - version 2.0
→ Authentication timeout: 120 secs; Authentication retries: 3

SW1# show crypto key mypubkey rsa
→ Affiche la clé publique RSA avec son nom (ex. SW1.lab.lan)

SW1# show users
→ Affiche les sessions actives (console + VTY)
```

### Test depuis PC-ADMIN (Packet Tracer)

Dans le **Telnet/SSH Client** du PC-ADMIN :

| Paramètre | Valeur |
|---|---|
| Connection Type | SSH |
| Host | 192.168.99.254 |
| Username | admin |
| Password | Azerty1* |

→ L'accès direct au mode privilégié (niveau 15) confirme l'authentification correcte.

> **Rappel :** En SSH le flux est **chiffré** dès la phase de négociation. En Telnet (TCP/23) tout circule **en clair**.

---

## 10. Cheat sheet complète — Mise en place d'un lab sécurisé

### Exemple : SW1 (VLAN 10 + VLAN 99 admin + DEAD_ZONE + SSH)

```
SW1> enable
SW1# configure terminal

! --- Nommage des VLANs ---
SW1(config)# vlan 10
SW1(config-vlan)# name MARKETING
SW1(config-vlan)# exit
SW1(config)# vlan 20
SW1(config-vlan)# name RH
SW1(config-vlan)# exit
SW1(config)# vlan 99
SW1(config-vlan)# name ADMIN
SW1(config-vlan)# exit
SW1(config)# vlan 666
SW1(config-vlan)# name DEAD_ZONE
SW1(config-vlan)# exit

! --- Ports access VLAN 10 ---
SW1(config)# interface range fastEthernet 0/1 - 6
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport access vlan 10
SW1(config-if-range)# exit

! --- Ports access VLAN 20 ---
SW1(config)# interface range fastEthernet 0/13 - 18
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport access vlan 20
SW1(config-if-range)# exit

! --- Port admin PC-ADMIN ---
SW1(config)# interface fastEthernet 0/24
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 99
SW1(config-if)# exit

! --- DEAD_ZONE : ports inutilisés ---
SW1(config)# interface range fastEthernet 0/7 - 12
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport access vlan 666
SW1(config-if-range)# shutdown
SW1(config-if-range)# exit
SW1(config)# interface range fastEthernet 0/19 - 23
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport access vlan 666
SW1(config-if-range)# shutdown
SW1(config-if-range)# exit

! --- Trunk vers SW2 ---
SW1(config)# interface gigabitEthernet 0/1
SW1(config-if)# switchport mode trunk
SW1(config-if)# exit

! --- SVI d'administration ---
SW1(config)# interface vlan 99
SW1(config-if)# ip address 192.168.99.254 255.255.255.0
SW1(config-if)# no shutdown
SW1(config-if)# exit

! --- Désactivation VLAN 1 ---
SW1(config)# interface vlan 1
SW1(config-if)# no ip address
SW1(config-if)# shutdown
SW1(config-if)# exit

! --- SSH ---
SW1(config)# ip domain-name lab.lan
SW1(config)# username admin privilege 15 secret Azerty1*
SW1(config)# crypto key generate rsa modulus 4096
SW1(config)# ip ssh version 2
SW1(config)# line vty 0 4
SW1(config-line)# login local
SW1(config-line)# transport input ssh
SW1(config-line)# exec-timeout 5 0
SW1(config-line)# exit

! --- Sauvegarde ---
SW1(config)# end
SW1# write
```

---

## 11. Commandes de vérification — Référence rapide

| Commande | Ce qu'elle affiche |
|---|---|
| `show vlan brief` | VLANs et ports associés |
| `show vlan` | Détail complet des VLANs |
| `show interfaces status` | Statut de tous les ports (connecté/disabled/VLAN) |
| `show ip interface brief` | Résumé IP + état up/down de chaque interface |
| `show interfaces trunk` | Ports trunk actifs et VLANs autorisés |
| `show running-config` | Configuration courante (en mémoire vive) |
| `show startup-config` | Configuration de démarrage (en NVRAM) |
| `show ip ssh` | Statut SSH et version |
| `show crypto key mypubkey rsa` | Clé publique RSA générée |
| `show users` | Sessions actives (console + VTY) |
| `show interfaces fastEthernet 0/1` | Détail complet d'un port |
| `show interfaces vlan 99` | Statut de l'interface SVI |

---

*Document basé sur les labs demo-reseau-vlan-revision-1 à 4 — Packet Tracer / Cisco 2960*
