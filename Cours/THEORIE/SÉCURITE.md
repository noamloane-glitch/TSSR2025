
---

## 🛡️ INTRODUCTION À LA CYBERSÉCURITÉ

**Q1 : Que signifie D.I.C.P ?**

> [!success]- 🔓 Réponse
> Disponibilité, Intégrité, Confidentialité, Preuve (traçabilité)

---

**Q2 : Différence entre Sécurité et Sûreté ?**

> [!success]- 🔓 Réponse
> Sécurité = protection contre actions malveillantes / Sûreté = protection contre dysfonctionnements et accidents

---
---

## 🔐 CRYPTOGRAPHIE

### Question 1
**Différence entre chiffrement symétrique et asymétrique ?**

> [!success]- 🔓 Réponse
> - **Symétrique** : 1 seule clé pour chiffrer ET déchiffrer — rapide
> - **Asymétrique** : 2 clés (publique + privée) — lent mais sécurisé

---

### Question 2
**C'est quoi une fonction de hachage ?**

> [!success]- 🔓 Réponse
> Fonction qui génère une **empreinte unique** de taille fixe à partir d'un message
> **Irréversible** : impossible de retrouver le message original
> Ex : SHA-256, MD5

---

### Question 3
**Pourquoi 2 messages de tailles différentes donnent un hash SHA-512 de même longueur ? (CP4 Q6.2)**

> [!success]- 🔓 Réponse
> Car une fonction de hachage produit **toujours une sortie de taille fixe**. SHA-512 = **512 bits** (128 caractères hexa), quelle que soit la taille de l'entrée.

---

### Question 4
**Alice veut envoyer un message secret à Bob. Quelles clés utiliser ? (CP4 Q6.5)**

> [!success]- 🔓 Réponse
> Alice chiffre avec la **clé publique de Bob** → Bob déchiffre avec sa **clé privée** → chiffrement **asymétrique**

---

### Question 5
**Pourquoi utilise-t-on la cryptographie hybride ?**

> [!success]- 🔓 Réponse
> - **Asymétrique** pour échanger la clé de session de façon sécurisée
> - **Symétrique** pour chiffrer les données (plus rapide)
> = combine sécurité + performance

---

### Question 6
**C'est quoi une signature numérique ?**

> [!success]- 🔓 Réponse
> **Hachage** du message + **chiffrement** du hash avec la clé privée
> Garantit : **authentification** (c'est bien l'émetteur) + **intégrité** (message non modifié)

---

### Question 7
**Quels algorithmes de hachage sont obsolètes ?**

> [!success]- 🔓 Réponse
> **MD5** et **SHA-1** — vulnérables aux collisions
> Utiliser **SHA-256** ou **SHA-3** minimum

---
---

## 🔐 SSH (Secure Shell)

**Que signifie TOFU ?**

> [!success]- 🔓 Réponse
> Trust On First Use — À la première connexion, on fait confiance à la clé reçue

---
---

## 🔐 VPN (Virtual Private Network)

### Question 1
**C'est quoi un VPN ?**

> [!success]- 🔓 Réponse
> Tunnel **chiffré** pour se connecter à un réseau distant de façon sécurisée.

---

### Question 2
**Différence VPN site-à-site et nomade ? (Question CP4)**

> [!success]- 🔓 Réponse
> - **Site-à-site** = 2 réseaux reliés (permanent, transparent)
> - **Nomade** = 1 user se connecte au réseau entreprise

---

### Question 3
**C'est quoi IPsec ?**

> [!success]- 🔓 Réponse
> Protocole VPN niveau 3 (IP). AH = intégrité, ESP = intégrité + **chiffrement**.

---

### Question 4
**C'est quoi OpenVPN ?**

> [!success]- 🔓 Réponse
> Solution VPN libre basée sur **TLS**, port 1194 (UDP ou TCP).

---

### Question 5
**Différence mode tunnel et transport (IPsec) ?**

> [!success]- 🔓 Réponse
> - **Tunnel** = protège tout le paquet (site-à-site)
> - **Transport** = protège les données seulement

---

### Question 6
**Un VPN est-il 100% sécurisé ?**

> [!success]- 🔓 Réponse
> Non, un VPN ouvre une **brèche** entre 2 réseaux. Il faut authentification robuste + surveillance.

---

## 📋 Types de VPN (Question CP4 Q6.4)

| Type | C'est quoi | Exemple |
| ---- | ---------- | ------- |
|      |            |         |
|      |            |         |
|      |            |         |

> [!success]- 🔓 Réponse
>
> | Type | C'est quoi | Exemple |
> | ---- | ---------- | ------- |
> | **Site-à-site** | Relie 2 réseaux d'entreprise (permanent, transparent pour les users) | IPsec tunnel entre 2 routeurs |
> | **Nomade (remote access)** | 1 utilisateur distant se connecte au réseau entreprise | OpenVPN, WireGuard depuis un laptop |
> | **SSL/TLS** | VPN via navigateur web, sans client lourd | Portail web entreprise (clientless) |

---

## 🔒 SÉCURISER LES SYSTÈMES

### Question 1
**Qu'est-ce que le principe de minimisation ?**

> [!success]- 🔓 Réponse
> Réduire la **surface d'attaque** en ne gardant que les composants et services strictement nécessaires. Cela facilite les mises à jour, le suivi et la surveillance du système.

---

### Question 2
**Qu'est-ce que le hardening ?**

> [!success]- 🔓 Réponse
> Le **durcissement** de la configuration d'un système. La configuration par défaut n'est pas satisfaisante (ex : mots de passe par défaut). Il faut appliquer les recommandations de l'éditeur ou de l'ANSSI.

---

### Question 3
**Qu'est-ce que le principe de moindre privilège ? Comment l'appliquer ?**

> [!success]- 🔓 Réponse
> Donner uniquement les droits **strictement nécessaires** à chaque utilisateur. Application : désactiver comptes inutiles, interdire comptes partagés, utiliser **sudo** pour l'élévation temporaire, administrateurs = plusieurs comptes (un admin + un standard).

---

### Question 4
**Qu'est-ce que la défense en profondeur ?**

> [!success]- 🔓 Réponse
> Principe consistant à mettre **autant de barrières que possible** pour retarder/dissuader un attaquant. Le firewall réseau seul ne suffit pas car les services ouverts peuvent être vulnérables et l'attaquant peut être interne.

---

### Question 5 ⭐ CP4 6.7
**En matière de sécurité informatique, indique 3 types de menaces auxquelles peut être confronté un SI.**

> [!success]- 🔓 Réponse
> 1. **Exploitation de failles** (vulnérabilités 0-day, failles logicielles type EternalBlue/WannaCry)
> 2. **Logiciels malveillants** (virus, ransomware, malwares)
> 3. **Attaques réseau** (intrusions, rebond interne, attaques depuis l'extérieur)