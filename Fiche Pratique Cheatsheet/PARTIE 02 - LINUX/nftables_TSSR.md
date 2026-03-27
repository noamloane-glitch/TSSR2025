# nftables — Pare-feu Linux

> Successeur moderne de `iptables`. Outil de filtrage réseau intégré au noyau Linux.  
> Par défaut sur **Debian 10+**, **Ubuntu 20.04+**, **RHEL 8+**.

---

## Concept & Couche OSI

| Outil | Couche OSI | Type de filtrage | Rôle |
|-------|-----------|-----------------|------|
| `nftables` | 3-4 (+ 7 avec modules) | Stateful / Stateless | Pare-feu Linux moderne |

> Remplace : `iptables`, `ip6tables`, `arptables`, `ebtables` — **tout en un seul outil**.

---

## Concepts clés : Tables, Chaînes, Règles

```
Table (famille)
 └── Chaîne (hook)
      └── Règle (critère → action)
```

### Familles de tables

| Famille | Trafic concerné |
|---------|----------------|
| `ip` | IPv4 uniquement |
| `ip6` | IPv6 uniquement |
| `inet` | ✅ IPv4 + IPv6 (recommandé) |
| `arp` | ARP |
| `bridge` | Pont réseau |

### Hooks (points d'accroche)

| Hook | Moment d'interception |
|------|----------------------|
| `prerouting` | Avant routage (tout trafic entrant) |
| `input` | Trafic à destination de la machine |
| `forward` | Trafic routé (traverse la machine) |
| `output` | Trafic généré par la machine |
| `postrouting` | Après routage (tout trafic sortant) |

### Actions (verdicts)

| Action | Effet |
|--------|-------|
| `accept` | Autorise le paquet |
| `drop` | Rejette silencieusement |
| `reject` | Rejette avec notification ICMP |
| `log` | Journalise sans bloquer |
| `return` | Remonte à la chaîne parente |

---

## Procédure : Installation & Activation

```bash
# Installer nftables
sudo apt install nftables

# Activer et démarrer
sudo systemctl enable nftables
sudo systemctl start nftables

# Vérifier l'état
sudo systemctl status nftables

# Fichier de configuration principal
sudo nano /etc/nftables.conf

# Appliquer une configuration
sudo nft -f /etc/nftables.conf

# Sauvegarder les règles actuelles
sudo nft list ruleset > /etc/nftables.conf
```

---

## Commandes essentielles

| Commande | Rôle |
|----------|------|
| `nft list ruleset` | Afficher toutes les règles |
| `nft list tables` | Lister les tables |
| `nft list table inet filter` | Afficher une table |
| `nft flush ruleset` | ⚠️ Supprimer TOUTES les règles |
| `nft add table inet filter` | Créer une table |
| `nft add chain inet filter input` | Créer une chaîne |
| `nft add rule inet filter input ...` | Ajouter une règle en **fin** de chaîne |
| `nft insert rule inet filter input ...` | Insérer une règle en **début** de chaîne |
| `nft add rule inet filter input position <h> ...` | Insérer **après** le handle `<h>` |
| `nft insert rule inet filter input position <h> ...` | Insérer **avant** le handle `<h>` |
| `nft --handle list chain inet filter input` | Afficher les règles **avec leurs handles** |
| `nft delete rule inet filter input handle 3` | Supprimer une règle par handle |
| `nft -f /etc/nftables.conf` | Charger un fichier de règles |
| `nft list ruleset > sauvegarde.nft` | Sauvegarder les règles |

---

## Exemple de configuration type

```bash
#!/usr/sbin/nft -f

# Vider les règles existantes
flush ruleset

# Créer une table inet (IPv4 + IPv6)
table inet filter {

    # Chaîne INPUT — trafic entrant
    chain input {
        type filter hook input priority 0; policy drop;

        # Loopback autorisé
        iif lo accept

        # Connexions établies et liées autorisées (stateful)
        ct state established,related accept

        # Rejeter les paquets invalides
        ct state invalid drop

        # ICMP/ping autorisé
        ip protocol icmp accept
        ip6 nexthdr icmpv6 accept

        # SSH autorisé
        tcp dport 22 accept

        # HTTP/HTTPS autorisé
        tcp dport { 80, 443 } accept
    }

    # Chaîne OUTPUT — trafic sortant
    chain output {
        type filter hook output priority 0; policy accept;
    }

    # Chaîne FORWARD — trafic routé
    chain forward {
        type filter hook forward priority 0; policy drop;
    }
}
```

---

## Procédure : Insérer une règle à une position spécifique

### Étape 1 — Afficher les handles (numéros de règles)

```bash
# ⚠️ Sans --handle, les positions ne sont pas visibles
sudo nft --handle list chain inet filter input
```

Sortie exemple :
```
table inet filter {
    chain input {
        type filter hook input priority 0; policy drop;
        iif lo accept                          # handle 2
        ct state established,related accept    # handle 3
        ct state invalid drop                  # handle 4
        tcp dport 22 accept                    # handle 5
        tcp dport { 80, 443 } accept           # handle 6
    }
}
```

### Étape 2 — Choisir la méthode d'insertion

| Commande | Comportement | Positionnement |
|----------|-------------|----------------|
| `nft add rule ...` | Ajoute à la **fin** de la chaîne | Après toutes les règles |
| `nft insert rule ...` | Insère au **début** de la chaîne | Avant toutes les règles |
| `nft add rule ... position <handle>` | Insère **après** le handle indiqué | Position précise |
| `nft insert rule ... position <handle>` | Insère **avant** le handle indiqué | Position précise |

### Étape 3 — Insérer à une position précise

```bash
# Insérer AVANT le handle 5 (avant la règle SSH)
sudo nft insert rule inet filter input position 5 tcp dport 8080 accept

# Insérer APRÈS le handle 4 (après la règle invalid drop)
sudo nft add rule inet filter input position 4 tcp dport 8080 accept

# Insérer au tout début de la chaîne
sudo nft insert rule inet filter input tcp dport 8080 accept

# Insérer à la fin de la chaîne
sudo nft add rule inet filter input tcp dport 8080 accept
```

### Étape 4 — Vérifier le résultat

```bash
sudo nft --handle list chain inet filter input
```

### Étape 5 — Sauvegarder pour persistance

```bash
# Sauvegarder dans le fichier de config
sudo nft list ruleset > /etc/nftables.conf
```

> ⚠️ Sans sauvegarde dans `/etc/nftables.conf`, les règles sont **perdues au redémarrage**.

---

## nftables vs iptables

| Aspect | `iptables` | `nftables` |
|--------|-----------|-----------|
| **Protocoles** | 1 outil par famille | ✅ Un seul outil |
| **Syntaxe** | Complexe, verbeuse | ✅ Plus lisible |
| **Performance** | Bonne | ✅ Meilleure |
| **Ensembles (sets)** | Limité | ✅ Natif (`{ 80, 443 }`) |
| **Compteurs** | Externes | ✅ Intégrés |
| **Compatibilité** | Partout | Debian 10+, Ubuntu 20.04+ |

---

## Fichiers importants

| Fichier | Rôle |
|---------|------|
| `/etc/nftables.conf` | Configuration principale (chargée au démarrage) |
| `/etc/nftables.d/` | Dossier de configs modulaires (si activé) |

---

## ⚠️ À retenir absolument

- `nftables` = successeur de `iptables` — même concept, syntaxe différente
- Structure : **Table → Chaîne → Règle** (dans cet ordre)
- Famille `inet` = IPv4 **et** IPv6 simultanément → à préférer
- `policy drop` = tout ce qui n'est pas explicitement autorisé est bloqué (**whitelist**)
- `ct state established,related accept` = indispensable pour le mode **stateful**
- `nft flush ruleset` supprime **tout** sans confirmation — dangereux !
- Toujours sauvegarder avant de modifier : `nft list ruleset > backup.nft`
- Couche OSI : **3-4** (réseau + transport)
