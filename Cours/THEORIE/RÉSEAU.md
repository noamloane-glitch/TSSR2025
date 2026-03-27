
---

## IPV4

### **Contexte**
L'entreprise TechCorp a obtenu l'adresse réseau suivante :
**192.168.50.0/24**
Elle doit créer 4 sous-réseaux pour :
- Sous-réseau A : Direction (15 hôtes)
- Sous-réseau B : Comptabilité (30 hôtes)
- Sous-réseau C : Production (60 hôtes)
- Sous-réseau D : Informatique (10 hôtes)

---

## 📝 QUESTIONS À RÉSOUDRE

### **PARTIE 1 : Analyse du réseau principal**
**192.168.50.0/24**

1. Convertir l'adresse IP en binaire =
2. Quel est le masque en notation décimale ?
3. Quelle est l'adresse réseau ?
4. Quelle est l'adresse de broadcast ?
5. Combien d'hôtes utilisables au total ?
6. Quelle est la plage d'adresses utilisables ?
7. Quelle est la classe de cette adresse ?

### **PARTIE 1A : Découpage SYMÉTRIQUE**
Diviser le réseau en 4 sous-réseaux égaux et attribuer à chaque département.

---

### **PARTIE 2 : Sous-réseaux (VLSM)**
Pour chaque sous-réseau, déterminer :

**Sous-réseau C (Production - 60 hôtes) :**
8. Masque nécessaire (CIDR et décimal)
9. Adresse réseau
10. Première IP utilisable
11. Dernière IP utilisable
12. Adresse de broadcast
13. Nombre d'hôtes utilisables

**Sous-réseau B (Comptabilité - 30 hôtes) :**
14-19. Mêmes questions

**Sous-réseau A (Direction - 15 hôtes) :**
20-25. Mêmes questions

**Sous-réseau D (Informatique - 10 hôtes) :**
26-31. Mêmes questions

---

### **PARTIE 3 : Exercices pratiques**

32. Un PC a l'IP **192.168.50.75/27**. Donner :
    - Adresse réseau
    - Broadcast
    - Plage utilisable

33. Deux machines peuvent-elles communiquer directement ?
    - PC1 : 192.168.50.45/26
    - PC2 : 192.168.50.80/26

34. Convertir en binaire : **192.168.50.130**

35. Quel masque pour 500 hôtes minimum ?

---

## Types de diffusion IPv4

| Type          | Quoi ? | Quand ? (exemple) | @IP (exemple) | Matériel concerné |
| ------------- | ------ | ----------------- | ------------- | ----------------- |
| **Unicast**   |        |                   |               |                   |
| **Multicast** |        |                   |               |                   |
| **Broadcast** |        |                   |               |                   |
| **Anycast**   |        |                   |               |                   |

> [!success]- 🔓 Réponse
>
> | Type          | Quoi ?                                               | Quand ? (exemple)                                                 | @IP (exemple)                                                    | Matériel concerné     |
> | ------------- | ---------------------------------------------------- | ----------------------------------------------------------------- | ---------------------------------------------------------------- | --------------------- |
> | **Unicast**   | 1 → 1                                                | Ping, HTTP, SSH, toute communication classique                    | `192.168.10.10/24`                                               | Tout matériel IP      |
> | **Multicast** | 1 → plusieurs (groupe)                               | Streaming, visioconférence, meeting, protocoles de routage (OSPF) | `224.0.0.0` → `239.255.255.255` (classe D)                       | PC, routeurs, switchs |
> | **Broadcast** | 1 → tous les nœuds du réseau/VLAN                    | DHCPDISCOVER, ARP Request                                         | `255.255.255.255` (local) / `192.168.10.255` (dirigé sur /24)    | PC, switchs           |
> | **Anycast**   | 1 → le plus proche parmi plusieurs ayant la même @IP | Serveurs DNS racine (13 domaines, +130 serveurs physiques), CDN   | Même IP attribuée à plusieurs serveurs géographiquement répartis | Routeurs, serveurs    |

---

## IPV6

## Types de diffusion IPv6

| Type              | Quoi ? | Quand ? (exemple) | @IPv6 (exemple) | Différence avec IPv4 |
| ----------------- | ------ | ----------------- | --------------- | -------------------- |
| **Unicast**       | 1 : 1  |                   |                 |                      |
| **Multicast**     |        |                   |                 |                      |
| **Anycast**       |        |                   |                 |                      |
| **~~Broadcast~~** |        |                   |                 |                      |

> [!success]- 🔓 Réponse
>
> | Type              | Quoi ?                                               | Quand ? (exemple)                                                 | @IPv6 (exemple)                                                  | Différence avec IPv4 |
> | ----------------- | ---------------------------------------------------- | ----------------------------------------------------------------- | ---------------------------------------------------------------- | --------------------- |
> | **Unicast**       | 1 → 1                                                | Ping6, HTTP, SSH, toute communication classique                   | `2001:db8::1/64` (global) / `fe80::1` (link-local)              | Pareil qu'IPv4 |
> | **Multicast**     | 1 → plusieurs (groupe)                               | Découverte de voisins (NDP), routage (OSPFv3), streaming          | `ff02::1` (tous les nœuds) / `ff02::2` (tous les routeurs)      | Remplace aussi le broadcast |
> | **Anycast**       | 1 → le plus proche parmi plusieurs ayant la même @IP | Serveurs DNS racine, CDN, passerelle par défaut                   | Même adresse unicast attribuée à plusieurs interfaces            | Pareil qu'IPv4 |
> | **~~Broadcast~~** | ~~1 → tous~~                                         | —                                                                 | —                                                                | **N'EXISTE PAS en IPv6** → remplacé par multicast `ff02::1` |

---

## Les types d'adresses IPv6

| Préfixe | Type | Scope |
| ------- | ---- | ----- |
|         |      |       |
|         |      |       |
|         |      |       |
|         |      |       |
|         |      |       |

> [!success]- 🔓 Réponse
>
> | Préfixe | Type | Scope |
> | ------- | ---- | ----- |
> | `fe80::/10` | **Link-Local** | Lien local uniquement (non routable) |
> | `fc00::/7` (fd00::/8) | **Unique Local (ULA)** | Réseau privé (équivalent IPv4 privé) |
> | `2000::/3` | **Global Unicast (GUA)** | Internet (routable mondialement) |
> | `ff00::/8` | **Multicast** | Groupe (remplace aussi le broadcast) |
> | `::1/128` | **Loopback** | Machine locale (équivalent 127.0.0.1) |

---

## NAT/PAT, DNAT, SNAT

## Les types de NAT


| Type | Nom complet | Sens | Usage |
| ---- | ----------- | ---- | ----- |
|      |             |      |       |
|      |             |      |       |
|      |             |      |       |
|      |             |      |       |
|      |             |      |       |

> [!success]- 🔓 Réponse
>
> | Type                | Nom complet              | Sens           | Usage                                               |
> | ------------------- | ------------------------ | -------------- | --------------------------------------------------- |
> | **PAT/NAPT**        | Port Address Translation | Source         | Sortie Internet (plusieurs clients → 1 IP publique) |
> | **SNAT**            | Source NAT               | Source         | Sortie Internet                                     |
> | **DNAT**            | Destination NAT          | Destination    | Publier un serveur interne                          |
> | **NAT 1:1**         | Static NAT               | Bidirectionnel | 1 IP privée = 1 IP publique dédiée                  |
> | **Port forwarding** | Redirection de port      | Destination    | Exposer un service sur un port précis               |

---

## 📝 Autres noms du PAT/NAPT

| Nom | Contexte |
| --- | -------- |
|     |          |
|     |          |
|     |          |

> [!success]- 🔓 Réponse
>
> | Nom | Contexte |
> | --- | -------- |
> | NAT overload | Cisco |
> | NAT masquerade | Linux (iptables/nftables) |
> | SNAT avec ports | pfSense, Stormshield |

---


---

## VOIP

**Quelle est la différence entre RTC, VoIP et ToIP ?**


> [!success]- 🔓 Réponse
> - **RTC** (Réseau Téléphonique Commuté) = technologie historique basée sur un réseau cuivre pour la téléphonie fixe. Arrêt programmé entre 2023 et 2030.
> - **VoIP** (Voice over IP) = technologie/protocole permettant de transmettre la voix sur des réseaux IP (LAN, WAN), filaires ou non. VoIP = un protocole.
> - **ToIP** (Telephony over IP) = gestion de réseaux de téléphonie locale à l'aide d'équipements spécifiques (IPBX, terminaux IP…). ToIP = une infrastructure. Un réseau ToIP s'appuie sur la VoIP pour fonctionner.

---

**DIAGNOSTIC : Un softphone ne peut pas passer d'appels. Que vérifies-tu ?**



> [!success]- 🔓 Réponse
> 1. Le softphone affiche-t-il **"Registered"** ? Si "Registration failed" → vérifier username, password, IP du serveur
> 2. Le serveur IPBX est-il joignable ? → `ping 172.16.10.5`
> 3. Le port SIP **5060 UDP** est-il ouvert ? (pare-feu)
> 4. L'extension existe-t-elle dans FreePBX ? A-t-on cliqué **"Apply Config"** ?
> 5. Pas de son → vérifier les périphériques audio dans le softphone

---

**Qu'est-ce que le protocole SIP ? Sur quel port fonctionne-t-il ?**

> [!success]- 🔓 Réponse
> **SIP** (Session Initiation Protocol) — protocole de signalisation défini par l'IETF (RFC 3261). Protocole ouvert permettant d'établir et contrôler des sessions multimédia sur un réseau IP. Utilisé pour la ToIP/VoIP, visioconférence, IoT. Port **5060** (UDP/TCP) et **5061** (TLS/sécurisé).