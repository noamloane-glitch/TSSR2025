
## 📚 Table des matières

- [Introduction à la table filter](#introduction-à-la-table-filter)
- [Structure de la table filter](#structure-de-la-table-filter)
- [Les trois chaînes principales](#les-trois-chaînes-principales)
  - [Chaîne INPUT](#chaîne-input)
  - [Chaîne OUTPUT](#chaîne-output)
  - [Chaîne FORWARD](#chaîne-forward)
- [Syntaxe des règles iptables](#syntaxe-des-règles-iptables)
- [Critères de correspondance](#critères-de-correspondance)
  - [Filtrage par adresse source](#filtrage-par-adresse-source)
  - [Filtrage par adresse destination](#filtrage-par-adresse-destination)
  - [Filtrage par protocole](#filtrage-par-protocole)
  - [Filtrage par port](#filtrage-par-port)
- [Actions (targets)](#actions-targets)
- [Ordre d'évaluation des règles](#ordre-dévaluation-des-règles)
- [Pièges courants](#pièges-courants)
- [Bonnes pratiques](#bonnes-pratiques)

---

## Introduction à la table filter

La table **filter** est la table par défaut et la plus utilisée dans Netfilter/iptables. Son rôle principal est de **décider si un paquet doit être autorisé ou bloqué**.

> [!info] Pourquoi la table filter ?
> - C'est la table utilisée par défaut si vous ne spécifiez pas `-t`
> - Elle contient toute la logique de filtrage du pare-feu
> - Elle est sollicitée pour chaque paquet traversant le système

> [!warning] À ne pas confondre
> La table filter ne modifie PAS les paquets (pas de NAT, pas de modification de contenu). Elle décide uniquement s'ils passent ou non.

---

## Structure de la table filter

La table filter contient **3 chaînes prédéfinies** qui correspondent à différents points de passage des paquets :

```
                    ┌─────────────┐
                    │   MACHINE   │
                    └─────────────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
         ┌────▼───┐   ┌────▼────┐   ┌──▼──────┐
         │ INPUT  │   │ FORWARD │   │ OUTPUT  │
         └────────┘   └─────────┘   └─────────┘
              │            │            │
     Paquets entrants  Routés via   Paquets sortants
     vers la machine   la machine   de la machine
```

> [!tip] Astuce mnémotechnique
> - **INPUT** : ce qui entre DANS votre système
> - **OUTPUT** : ce qui sort DE votre système
> - **FORWARD** : ce qui passe À TRAVERS votre système

---

## Les trois chaînes principales

### Chaîne INPUT

La chaîne INPUT traite **tous les paquets destinés à la machine locale**.

**Quand utiliser INPUT :**
- Protéger les services locaux (SSH, HTTP, base de données)
- Limiter les connexions entrantes
- Bloquer des attaques ciblant vos services

```bash
# Exemple : autoriser SSH uniquement depuis un réseau spécifique
iptables -A INPUT -p tcp --dport 22 -s 192.168.1.0/24 -j ACCEPT

# Exemple : bloquer tout le reste par défaut
iptables -P INPUT DROP
```

> [!example] Cas d'usage typique
> Un serveur web doit accepter le trafic HTTP/HTTPS (ports 80/443) mais refuser tout autre accès non sollicité.

---

### Chaîne OUTPUT

La chaîne OUTPUT traite **tous les paquets générés par la machine locale**.

**Quand utiliser OUTPUT :**
- Contrôler les connexions sortantes d'un serveur
- Empêcher des programmes malveillants de communiquer
- Restreindre l'accès Internet des utilisateurs/services

```bash
# Exemple : autoriser uniquement les requêtes DNS sortantes
iptables -A OUTPUT -p udp --dport 53 -j ACCEPT

# Exemple : bloquer les connexions vers un réseau spécifique
iptables -A OUTPUT -d 10.0.0.0/8 -j DROP
```

> [!warning] Attention
> Soyez prudent avec OUTPUT ! Bloquer trop de trafic sortant peut casser des services système (DNS, NTP, mises à jour).

---

### Chaîne FORWARD

La chaîne FORWARD traite **les paquets qui traversent la machine** (routage).

**Quand utiliser FORWARD :**
- La machine agit comme routeur/passerelle
- Mise en place d'un pare-feu entre deux réseaux
- Contrôle du trafic inter-VLAN

```bash
# Exemple : autoriser le forwarding d'un réseau interne vers Internet
iptables -A FORWARD -s 192.168.1.0/24 -j ACCEPT

# Exemple : bloquer le forwarding entre deux réseaux
iptables -A FORWARD -s 10.0.1.0/24 -d 10.0.2.0/24 -j DROP
```

> [!info] Prérequis pour FORWARD
> Le forwarding IP doit être activé au niveau du kernel :
> ```bash
> echo 1 > /proc/sys/net/ipv4/ip_forward
> ```

---

## Syntaxe des règles iptables

### Structure générale

```bash
iptables [-t table] COMMANDE CHAÎNE [critères] -j ACTION
```

| Élément | Description | Exemples |
|---------|-------------|----------|
| `-t table` | Table cible (optionnel, défaut: filter) | `-t filter`, `-t nat` |
| `COMMANDE` | Action sur la règle | `-A` (append), `-I` (insert), `-D` (delete) |
| `CHAÎNE` | Chaîne ciblée | `INPUT`, `OUTPUT`, `FORWARD` |
| `critères` | Conditions de correspondance | `-s`, `-d`, `-p`, `--dport` |
| `-j ACTION` | Action à effectuer (jump/target) | `ACCEPT`, `DROP`, `REJECT` |

### Commandes principales

```bash
# Ajouter une règle à la fin de la chaîne
iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# Insérer une règle en première position
iptables -I INPUT 1 -p tcp --dport 443 -j ACCEPT

# Supprimer une règle spécifique
iptables -D INPUT -p tcp --dport 80 -j ACCEPT

# Lister toutes les règles
iptables -L -n -v

# Lister les règles avec numéros de ligne
iptables -L INPUT --line-numbers

# Supprimer une règle par numéro
iptables -D INPUT 3

# Vider toutes les règles d'une chaîne
iptables -F INPUT

# Définir la politique par défaut
iptables -P INPUT DROP
```

> [!tip] Options utiles pour lister
> - `-n` : affiche les IPs en numérique (plus rapide)
> - `-v` : mode verbeux (compteurs de paquets/octets)
> - `--line-numbers` : numéros de ligne (utile pour supprimer)

---

## Critères de correspondance

### Filtrage par adresse source

L'option `-s` (ou `--source`) filtre selon l'**adresse IP source**.

```bash
# Autoriser une IP spécifique
iptables -A INPUT -s 192.168.1.100 -j ACCEPT

# Autoriser un réseau entier (notation CIDR)
iptables -A INPUT -s 192.168.1.0/24 -j ACCEPT

# Bloquer une plage d'IPs
iptables -A INPUT -s 10.0.0.0/8 -j DROP

# Inverser la condition (tout SAUF cette source)
iptables -A INPUT ! -s 192.168.1.0/24 -j DROP
```

> [!example] Cas pratique
> Restreindre l'accès SSH à un réseau d'administration uniquement :
> ```bash
> iptables -A INPUT -p tcp --dport 22 -s 192.168.100.0/24 -j ACCEPT
> iptables -A INPUT -p tcp --dport 22 -j DROP
> ```

---

### Filtrage par adresse destination

L'option `-d` (ou `--destination`) filtre selon l'**adresse IP destination**.

```bash
# Bloquer l'accès vers un serveur spécifique
iptables -A OUTPUT -d 203.0.113.50 -j DROP

# Autoriser uniquement vers un réseau interne
iptables -A FORWARD -d 192.168.2.0/24 -j ACCEPT

# Bloquer tout sauf une destination
iptables -A OUTPUT ! -d 192.168.0.0/16 -j DROP
```

> [!warning] Piège courant
> Ne confondez pas :
> - `-s` = source (d'où vient le paquet)
> - `-d` = destination (où va le paquet)

---

### Filtrage par protocole

L'option `-p` (ou `--protocol`) filtre selon le **protocole réseau**.

**Protocoles courants :**
- `tcp` : TCP (connexion établie, fiable)
- `udp` : UDP (sans connexion, rapide)
- `icmp` : ICMP (ping, erreurs réseau)
- `all` : tous les protocoles (défaut)

```bash
# Filtrer TCP
iptables -A INPUT -p tcp -j ACCEPT

# Filtrer UDP
iptables -A INPUT -p udp --dport 53 -j ACCEPT

# Filtrer ICMP (ping)
iptables -A INPUT -p icmp -j ACCEPT

# Bloquer un protocole spécifique
iptables -A INPUT -p udp -j DROP
```

> [!info] Numéros de protocole
> Vous pouvez aussi utiliser le numéro de protocole :
> ```bash
> iptables -A INPUT -p 6 -j ACCEPT  # TCP = 6
> iptables -A INPUT -p 17 -j ACCEPT # UDP = 17
> ```

---

### Filtrage par port

Les options `--sport` (source port) et `--dport` (destination port) filtrent selon les **ports TCP/UDP**.

> [!warning] Prérequis
> Vous DEVEZ spécifier `-p tcp` ou `-p udp` avant d'utiliser les options de port.

```bash
# Port de destination (le plus courant)
iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# Port source
iptables -A INPUT -p tcp --sport 1024:65535 -j ACCEPT

# Plage de ports
iptables -A INPUT -p tcp --dport 8000:9000 -j ACCEPT

# Plusieurs ports spécifiques (avec module multiport)
iptables -A INPUT -p tcp -m multiport --dports 80,443,8080 -j ACCEPT

# Combinaison source + destination
iptables -A FORWARD -p tcp --sport 80 --dport 1024:65535 -j ACCEPT
```

**Ports courants :**

| Service | Port | Protocole |
|---------|------|-----------|
| SSH | 22 | TCP |
| DNS | 53 | UDP (+ TCP) |
| HTTP | 80 | TCP |
| HTTPS | 443 | TCP |
| SMTP | 25 | TCP |
| POP3 | 110 | TCP |
| IMAP | 143 | TCP |
| MySQL | 3306 | TCP |
| PostgreSQL | 5432 | TCP |

> [!tip] Astuce multiport
> Au lieu de créer 10 règles pour 10 ports, utilisez multiport :
> ```bash
> iptables -A INPUT -p tcp -m multiport --dports 22,80,443,3306 -j ACCEPT
> ```

---

## Actions (targets)

Les actions définissent **ce qui arrive au paquet** quand il correspond à une règle.

### Actions principales

| Action | Effet | Quand l'utiliser |
|--------|-------|------------------|
| `ACCEPT` | Autorise le paquet | Trafic légitime |
| `DROP` | Rejette silencieusement | Trafic malveillant, scans |
| `REJECT` | Rejette avec message d'erreur | Trafic non autorisé (plus poli) |
| `LOG` | Enregistre dans les logs | Debugging, audit |
| `RETURN` | Retourne à la chaîne parente | Chaînes personnalisées |

### ACCEPT vs DROP vs REJECT

```bash
# ACCEPT : laisse passer
iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# DROP : jette silencieusement (l'émetteur attend un timeout)
iptables -A INPUT -p tcp --dport 23 -j DROP

# REJECT : envoie une réponse d'erreur (l'émetteur est informé)
iptables -A INPUT -p tcp --dport 23 -j REJECT

# REJECT avec type de message personnalisé
iptables -A INPUT -p tcp --dport 23 -j REJECT --reject-with icmp-port-unreachable
```

> [!tip] DROP vs REJECT : lequel choisir ?
> - **DROP** : pour le trafic malveillant (scans, attaques)
>   - Avantage : cache l'existence du service
>   - Inconvénient : timeout long côté client
> - **REJECT** : pour le trafic légitime mais non autorisé
>   - Avantage : réponse immédiate, facilite le debug
>   - Inconvénient : révèle que le port est filtré

### LOG : journalisation

```bash
# Logger avant de dropper
iptables -A INPUT -p tcp --dport 23 -j LOG --log-prefix "TELNET BLOCKED: "
iptables -A INPUT -p tcp --dport 23 -j DROP

# Logger avec niveau de log
iptables -A INPUT -j LOG --log-level 4 --log-prefix "INPUT-DROP: "

# Limiter le logging pour éviter la saturation
iptables -A INPUT -m limit --limit 5/min -j LOG --log-prefix "INPUT: "
```

> [!warning] Attention aux logs
> Sans limitation, le logging peut :
> - Remplir le disque
> - Ralentir le système
> - Faciliter les attaques DoS
> 
> Utilisez toujours `-m limit` avec LOG !

---

## Ordre d'évaluation des règles

> [!info] Principe fondamental
> Les règles sont évaluées **dans l'ordre**, de haut en bas. Dès qu'une règle correspond, son action est appliquée.

```bash
# ❌ MAUVAIS ORDRE
iptables -A INPUT -j DROP          # Bloque tout
iptables -A INPUT -p tcp --dport 80 -j ACCEPT  # Jamais atteint !

# ✅ BON ORDRE
iptables -A INPUT -p tcp --dport 80 -j ACCEPT  # D'abord autoriser
iptables -A INPUT -j DROP          # Puis bloquer le reste
```

### Exemple complet avec ordre logique

```bash
# 1. Autoriser le loopback (toujours en premier)
iptables -A INPUT -i lo -j ACCEPT

# 2. Autoriser les connexions établies (performance)
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# 3. Règles spécifiques (du plus spécifique au plus général)
iptables -A INPUT -p tcp --dport 22 -s 192.168.1.0/24 -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# 4. Logger ce qui n'a pas matché
iptables -A INPUT -j LOG --log-prefix "INPUT-DROP: "

# 5. Politique par défaut en dernier
iptables -A INPUT -j DROP
```

> [!tip] Optimisation
> Placez les règles les plus fréquemment matchées en haut pour améliorer les performances.

---

## Pièges courants

### 1. Oublier le loopback

```bash
# ❌ Casse les applications locales
iptables -P INPUT DROP

# ✅ Toujours autoriser lo en premier
iptables -A INPUT -i lo -j ACCEPT
iptables -P INPUT DROP
```

### 2. Bloquer les connexions établies

```bash
# ❌ Empêche les réponses de revenir
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -P INPUT DROP

# ✅ Autoriser les connexions établies
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -P INPUT DROP
```

### 3. Oublier les deux sens de la communication

```bash
# ❌ Autorise la requête mais pas la réponse
iptables -A OUTPUT -p tcp --dport 80 -j ACCEPT
iptables -P OUTPUT DROP

# ✅ Autoriser aussi le retour
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A OUTPUT -p tcp --dport 80 -j ACCEPT
```

### 4. Confondre --sport et --dport

```bash
# ❌ Bloque le serveur web (mauvais port)
iptables -A INPUT -p tcp --sport 80 -j ACCEPT

# ✅ Le serveur ÉCOUTE sur --dport
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
```

### 5. Appliquer les règles sans sauvegarde

```bash
# ❌ Perdu au redémarrage
iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# ✅ Sauvegarder
# Debian/Ubuntu
iptables-save > /etc/iptables/rules.v4

# RedHat/CentOS
service iptables save
```

---

## Bonnes pratiques

### 1. Politique par défaut restrictive

```bash
# Définir DROP par défaut
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT  # Ou DROP si besoin de contrôle strict
```

### 2. Structure de règles recommandée

```bash
#!/bin/bash

# Vider les règles existantes
iptables -F
iptables -X

# Politiques par défaut
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Loopback
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

# Connexions établies
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Services autorisés
iptables -A INPUT -p tcp --dport 22 -j ACCEPT  # SSH
iptables -A INPUT -p tcp --dport 80 -j ACCEPT  # HTTP
iptables -A INPUT -p tcp --dport 443 -j ACCEPT # HTTPS

# Logging (limité)
iptables -A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables-INPUT: " --log-level 7

# Sauvegarder
iptables-save > /etc/iptables/rules.v4
```

### 3. Commenter vos règles

```bash
# Utilisez des commentaires inline
iptables -A INPUT -p tcp --dport 22 -j ACCEPT -m comment --comment "SSH access"
iptables -A INPUT -p tcp --dport 80 -j ACCEPT -m comment --comment "HTTP web server"
```

### 4. Tester avant d'appliquer en production

```bash
# Script d'auto-rollback (évite de vous bloquer)
#!/bin/bash
iptables [vos règles]
echo "Règles appliquées. Vous avez 30 secondes pour tester."
echo "Appuyez sur CTRL+C pour garder, sinon rollback automatique."
sleep 30
iptables-restore < /etc/iptables/rules.v4.backup
```

### 5. Documenter et versionner

- Gardez vos règles dans des scripts versionnés (Git)
- Documentez POURQUOI chaque règle existe
- Maintenez un changelog des modifications

### 6. Monitorer les règles

```bash
# Vérifier les compteurs
watch -n 2 'iptables -L -n -v'

# Analyser les logs
tail -f /var/log/syslog | grep iptables

# Vérifier les connexions actives
conntrack -L
```

---

> [!tip] 💡 Points clés à retenir
> 1. La table **filter** = filtrage (autoriser/bloquer)
> 2. **INPUT** (vers le système), **OUTPUT** (depuis le système), **FORWARD** (à travers)
> 3. L'**ordre des règles** est crucial
> 4. Toujours autoriser **lo** et **ESTABLISHED,RELATED**
> 5. **ACCEPT** vs **DROP** vs **REJECT** selon le contexte
> 6. Sauvegarder et documenter vos règles

