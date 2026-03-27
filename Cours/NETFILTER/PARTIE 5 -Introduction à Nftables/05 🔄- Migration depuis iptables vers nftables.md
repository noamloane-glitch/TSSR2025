## 📑 Table des matières

- [Introduction à la migration](#introduction-à-la-migration)
- [Outils de conversion](#outils-de-conversion)
- [Méthodologie de migration](#méthodologie-de-migration)
- [Cohabitation iptables/nftables](#cohabitation-iptablesnftables)
- [Exemples de conversion](#exemples-de-conversion)

---

## Introduction à la migration

La migration d'iptables vers nftables est une évolution naturelle des systèmes Linux modernes. Bien que iptables reste fonctionnel, nftables offre de nombreux avantages qui justifient la migration.

### Pourquoi migrer ?

| Critère | iptables | nftables |
|---------|----------|----------|
| **Performance** | Bonne | Meilleure (bytecode optimisé) |
| **Syntaxe** | Verbeux, répétitif | Concise, unifiée |
| **IPv4/IPv6** | Outils séparés | Famille `inet` unique |
| **Atomicité** | Commandes successives | Chargement atomique |
| **Extensibilité** | Modules kernel | Architecture modulaire |
| **Maintenance** | Support hérité | Développement actif |
| **Debugging** | Difficile | Outils intégrés (tracing) |

> [!info] Timing de la migration
> - nftables est **mature et stable** depuis Linux 4.18+ (2018)
> - Debian 10+, Ubuntu 20.04+, RHEL 8+ utilisent nftables par défaut
> - iptables est maintenant une **surcouche** (iptables-nft) sur nftables dans de nombreuses distributions
> - La migration peut se faire progressivement

### Quand migrer ?

**Situations idéales pour migrer :**
- ✅ Nouvelle installation de système
- ✅ Refonte complète de la politique de sécurité
- ✅ Besoin de performance accrue
- ✅ Configuration complexe devenue ingérable avec iptables
- ✅ Consolidation IPv4/IPv6

**Situations où reporter :**
- ⚠️ Système de production critique sans fenêtre de maintenance
- ⚠️ Scripts de monitoring/automatisation fortement couplés à iptables
- ⚠️ Équipe non formée à nftables
- ⚠️ Logiciels tiers qui gèrent automatiquement iptables (Docker ancien, libvirt, etc.)

> [!warning] Compatibilité des applications
> Certaines applications modifient directement iptables (Docker, Kubernetes, fail2ban, etc.). Vérifiez leur compatibilité nftables avant de migrer un système en production.

---

## Outils de conversion

### iptables-translate : conversion simple

L'outil `iptables-translate` convertit les commandes iptables en syntaxe nftables équivalente.

#### Installation

```bash
# Debian/Ubuntu
apt install iptables nftables

# RHEL/CentOS
yum install iptables nftables

# Les outils sont généralement inclus dans le package iptables
```

#### Utilisation de base

```bash
# Convertir une règle unique
iptables-translate -A INPUT -p tcp --dport 22 -j ACCEPT
# Résultat : nft add rule ip filter INPUT tcp dport 22 counter accept

# Convertir une règle avec options
iptables-translate -A INPUT -s 192.168.1.0/24 -p tcp --dport 80 -m state --state NEW -j ACCEPT
# Résultat : nft add rule ip filter INPUT ip saddr 192.168.1.0/24 tcp dport 80 ct state new counter accept

# Version IPv6
ip6tables-translate -A INPUT -p tcp --dport 22 -j ACCEPT
# Résultat : nft add rule ip6 filter INPUT tcp dport 22 counter accept
```

> [!tip] Utilisation pratique
> `iptables-translate` est parfait pour :
> - Comprendre la syntaxe nftables équivalente
> - Convertir quelques règles simples
> - Apprendre la correspondance entre les deux syntaxes

#### iptables-restore-translate : conversion de ruleset complet

Pour convertir un ensemble complet de règles :

```bash
# Sauvegarder la configuration iptables actuelle
iptables-save > /tmp/iptables-rules.txt

# Convertir vers nftables
iptables-restore-translate -f /tmp/iptables-rules.txt > /tmp/nftables-rules.nft

# Pour IPv6
ip6tables-save > /tmp/ip6tables-rules.txt
ip6tables-restore-translate -f /tmp/ip6tables-rules.txt >> /tmp/nftables-rules.nft

# Vérifier le résultat
cat /tmp/nftables-rules.nft

# Tester le chargement (sans appliquer)
nft -c -f /tmp/nftables-rules.nft

# Appliquer si OK
nft -f /tmp/nftables-rules.nft
```

> [!warning] Limites des outils de conversion
> Les outils de traduction ont des limitations :
> - Ne gèrent pas tous les modules iptables
> - Produisent du code pas toujours optimal
> - Ne consolident pas IPv4/IPv6 en famille `inet`
> - Ne tirent pas parti des fonctionnalités avancées (sets, maps)
> - Peuvent générer des règles redondantes

### Conversion manuelle : la méthode recommandée

Pour une migration de qualité, préférez une conversion manuelle qui permet d'optimiser :

```bash
# Au lieu d'une conversion littérale...
iptables-translate -A INPUT -s 10.0.0.1 -j ACCEPT
iptables-translate -A INPUT -s 10.0.0.2 -j ACCEPT
iptables-translate -A INPUT -s 10.0.0.3 -j ACCEPT
# → 3 règles nftables

# Optimisez avec un set
nft add set inet filter trusted_ips { type ipv4_addr \; }
nft add element inet filter trusted_ips { 10.0.0.1, 10.0.0.2, 10.0.0.3 }
nft add rule inet filter input ip saddr @trusted_ips accept
# → 1 règle + 1 set (plus performant, plus maintenable)
```

### Vérification de compatibilité

Avant de migrer, vérifiez ce qui utilise iptables :

```bash
# Lister les applications qui ont iptables ouvert
lsof | grep iptables

# Vérifier les processus qui modifient iptables
ps aux | grep -E "(iptables|firewalld|ufw|docker)"

# Identifier les services systemd dépendants
systemctl list-dependencies iptables.service
systemctl list-dependencies ip6tables.service

# Vérifier les scripts custom
find /etc -name "*.sh" -exec grep -l "iptables" {} \;
find /usr/local -name "*.sh" -exec grep -l "iptables" {} \;
```

---

## Méthodologie de migration

### Étape 1 : Préparation

#### Documentation de l'existant

```bash
# 1. Sauvegarder la configuration actuelle
iptables-save > /backup/iptables-$(date +%Y%m%d).rules
ip6tables-save > /backup/ip6tables-$(date +%Y%m%d).rules

# 2. Documenter les règles avec commentaires
iptables -L -n -v --line-numbers > /backup/iptables-annotated.txt
ip6tables -L -n -v --line-numbers > /backup/ip6tables-annotated.txt

# 3. Tester la connectivité AVANT migration
ping -c 3 8.8.8.8
curl -I https://www.google.com
# Documenter tous les flux réseau critiques
```

#### Création d'un environnement de test

```bash
# Créer une VM/conteneur identique pour tester
# Ou utiliser un système de test avec la même config réseau

# Installer nftables
apt install nftables

# Vérifier la version
nft --version
# nftables v1.0.5 ou supérieur recommandé
```

> [!tip] Environnement de test
> **Testez TOUJOURS sur un système non-critique d'abord** :
> - VM locale
> - Conteneur Docker/LXC
> - Serveur de développement
> 
> Ne testez jamais directement en production, même pour "une petite règle" !

### Étape 2 : Conversion

#### Analyse de la configuration iptables

```bash
# Identifier les éléments à convertir
iptables -S | grep -E "^\-A" | sort | uniq

# Détecter les modules utilisés
iptables -S | grep -oP "\-m \K\w+" | sort | uniq

# Compter les règles par chaîne
for chain in INPUT OUTPUT FORWARD; do
    echo "$chain: $(iptables -L $chain -n | grep -c "^")"
done
```

#### Conversion progressive par table

```bash
# 1. Commencer par la table FILTER (la plus importante)
iptables-save -t filter > /tmp/filter.rules
iptables-restore-translate -f /tmp/filter.rules > /tmp/filter.nft

# 2. Puis NAT
iptables-save -t nat > /tmp/nat.rules
iptables-restore-translate -f /tmp/nat.rules > /tmp/nat.nft

# 3. Puis MANGLE si utilisée
iptables-save -t mangle > /tmp/mangle.rules
iptables-restore-translate -f /tmp/mangle.rules > /tmp/mangle.nft

# 4. Combiner et optimiser manuellement
cat /tmp/filter.nft /tmp/nat.nft /tmp/mangle.nft > /tmp/combined.nft
```

#### Optimisation manuelle

Après conversion automatique, optimisez :

```bash
# AVANT (règles converties littéralement)
nft add rule ip filter INPUT ip saddr 192.168.1.10 tcp dport 22 accept
nft add rule ip filter INPUT ip saddr 192.168.1.11 tcp dport 22 accept
nft add rule ip filter INPUT ip saddr 192.168.1.12 tcp dport 22 accept
nft add rule ip6 filter INPUT ip6 saddr 2001:db8::10 tcp dport 22 accept
nft add rule ip6 filter INPUT ip6 saddr 2001:db8::11 tcp dport 22 accept

# APRÈS (optimisé avec set et famille inet)
nft add set inet filter ssh_allowed { type ipv4_addr \; }
nft add element inet filter ssh_allowed { 192.168.1.10, 192.168.1.11, 192.168.1.12 }
nft add set inet filter ssh_allowed_v6 { type ipv6_addr \; }
nft add element inet filter ssh_allowed_v6 { 2001:db8::10, 2001:db8::11 }
nft add rule inet filter input ip saddr @ssh_allowed tcp dport 22 accept
nft add rule inet filter input ip6 saddr @ssh_allowed_v6 tcp dport 22 accept
```

> [!tip] Opportunités d'optimisation
> Lors de la migration, profitez-en pour :
> - Regrouper IPv4/IPv6 avec la famille `inet`
> - Utiliser des sets pour les listes d'IPs
> - Utiliser des maps pour les redirections multiples
> - Simplifier les chaînes personnalisées
> - Supprimer les règles obsolètes

### Étape 3 : Tests

#### Test en environnement contrôlé

```bash
# 1. Charger la nouvelle configuration nftables
nft -f /tmp/nftables-final.nft

# 2. Vérifier le chargement
nft list ruleset

# 3. Tester la connectivité
# Test entrée (depuis un autre hôte)
nc -zv <IP_SERVEUR> 22    # SSH
nc -zv <IP_SERVEUR> 80    # HTTP
nc -zv <IP_SERVEUR> 443   # HTTPS

# Test sortie (depuis le serveur)
ping -c 3 8.8.8.8
curl -I https://www.google.com
dig @8.8.8.8 google.com

# Test forward (si routeur/firewall)
# Depuis un client du réseau interne
curl -I https://www.google.com
```

#### Tests de non-régression

```bash
# Script de test automatisé
cat > /tmp/test-firewall.sh << 'EOF'
#!/bin/bash
TESTS_OK=0
TESTS_FAIL=0

# Test 1 : SSH accessible
if nc -zv -w 2 localhost 22 2>&1 | grep -q succeeded; then
    echo "✓ SSH OK"
    ((TESTS_OK++))
else
    echo "✗ SSH FAIL"
    ((TESTS_FAIL++))
fi

# Test 2 : Connexion sortante HTTPS
if curl -s -o /dev/null -w "%{http_code}" --max-time 5 https://www.google.com | grep -q 200; then
    echo "✓ HTTPS sortant OK"
    ((TESTS_OK++))
else
    echo "✗ HTTPS sortant FAIL"
    ((TESTS_FAIL++))
fi

# Test 3 : Ping sortant
if ping -c 2 -W 2 8.8.8.8 > /dev/null 2>&1; then
    echo "✓ Ping sortant OK"
    ((TESTS_OK++))
else
    echo "✗ Ping sortant FAIL"
    ((TESTS_FAIL++))
fi

echo "============================="
echo "Tests OK: $TESTS_OK"
echo "Tests FAIL: $TESTS_FAIL"
EOF

chmod +x /tmp/test-firewall.sh
/tmp/test-firewall.sh
```

#### Monitoring pendant la migration

```bash
# Terminal 1 : Monitoring des connexions
watch -n 1 'ss -tunap | grep ESTABLISHED | wc -l'

# Terminal 2 : Monitoring des logs
tail -f /var/log/syslog | grep -E "(nft|DROP|REJECT)"

# Terminal 3 : Monitoring des compteurs nftables
watch -n 1 'nft list ruleset | grep counter'
```

### Étape 4 : Déploiement en production

#### Planification du déploiement

> [!warning] Checklist pré-déploiement
> - [ ] Backup de la configuration iptables actuelle
> - [ ] Tests réussis en environnement de test
> - [ ] Documentation de la nouvelle configuration
> - [ ] Plan de rollback prêt
> - [ ] Fenêtre de maintenance planifiée
> - [ ] Équipe disponible pour support
> - [ ] Accès console/IPMI disponible (éviter le lockout SSH)

#### Procédure de déploiement

```bash
#!/bin/bash
# Script de migration iptables → nftables

set -e  # Arrêter en cas d'erreur

echo "=== MIGRATION IPTABLES → NFTABLES ==="
echo "Début: $(date)"

# 1. Backup iptables
echo "[1/6] Backup de la configuration actuelle..."
iptables-save > /backup/iptables-$(date +%Y%m%d-%H%M%S).rules
ip6tables-save > /backup/ip6tables-$(date +%Y%m%d-%H%M%S).rules

# 2. Vérifier la configuration nftables
echo "[2/6] Vérification de la configuration nftables..."
nft -c -f /etc/nftables.conf || {
    echo "ERREUR: Configuration nftables invalide!"
    exit 1
}

# 3. Arrêter iptables
echo "[3/6] Arrêt des services iptables..."
systemctl stop iptables.service 2>/dev/null || true
systemctl stop ip6tables.service 2>/dev/null || true
systemctl disable iptables.service 2>/dev/null || true
systemctl disable ip6tables.service 2>/dev/null || true

# 4. Vider les règles iptables
echo "[4/6] Vidage des règles iptables..."
iptables -F
iptables -X
iptables -t nat -F
iptables -t nat -X
iptables -t mangle -F
iptables -t mangle -X
iptables -P INPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -P OUTPUT ACCEPT

ip6tables -F
ip6tables -X
ip6tables -t nat -F
ip6tables -t nat -X
ip6tables -t mangle -F
ip6tables -t mangle -X
ip6tables -P INPUT ACCEPT
ip6tables -P FORWARD ACCEPT
ip6tables -P OUTPUT ACCEPT

# 5. Charger nftables
echo "[5/6] Chargement de la configuration nftables..."
systemctl enable nftables.service
systemctl start nftables.service

# 6. Vérification
echo "[6/6] Vérification..."
nft list ruleset | head -20

echo "=== MIGRATION TERMINÉE ==="
echo "Fin: $(date)"
echo ""
echo "⚠️  TESTEZ IMMÉDIATEMENT LA CONNECTIVITÉ !"
echo "Si problème, exécutez le rollback: /backup/rollback.sh"
```

#### Script de rollback

```bash
#!/bin/bash
# Script de rollback en cas de problème

cat > /backup/rollback.sh << 'EOF'
#!/bin/bash
set -e

echo "=== ROLLBACK VERS IPTABLES ==="

# 1. Arrêter nftables
systemctl stop nftables.service
systemctl disable nftables.service

# 2. Vider nftables
nft flush ruleset

# 3. Restaurer iptables
LATEST_BACKUP=$(ls -t /backup/iptables-*.rules | head -1)
LATEST_BACKUP_V6=$(ls -t /backup/ip6tables-*.rules | head -1)

iptables-restore < "$LATEST_BACKUP"
ip6tables-restore < "$LATEST_BACKUP_V6"

# 4. Réactiver iptables
systemctl enable iptables.service
systemctl start iptables.service
systemctl enable ip6tables.service
systemctl start ip6tables.service

echo "=== ROLLBACK TERMINÉ ==="
echo "Configuration iptables restaurée depuis:"
echo "  - $LATEST_BACKUP"
echo "  - $LATEST_BACKUP_V6"
EOF

chmod +x /backup/rollback.sh
```

> [!tip] Recommandations pour le déploiement
> 1. **Gardez une connexion console active** (IPMI, KVM) en cas de lockout SSH
> 2. **Déployez en heures creuses** pour minimiser l'impact
> 3. **Préparez un timer de sécurité** : `at now + 10 minutes -f /backup/rollback.sh`
> 4. **Testez immédiatement** tous les flux critiques après migration
> 5. **Documentez les changements** pour l'équipe

### Étape 5 : Post-migration

#### Vérification de la configuration active

```bash
# Lister la configuration complète
nft list ruleset > /backup/nftables-$(date +%Y%m%d)-prod.nft

# Vérifier les services actifs
systemctl status nftables.service
systemctl status iptables.service  # Doit être inactif

# Vérifier qu'iptables est bien désactivé
iptables -L -n | head  # Doit montrer des chaînes vides ou erreur
```

#### Monitoring continu

```bash
# Surveiller les compteurs de règles
nft list ruleset | grep counter

# Surveiller les logs de drop
journalctl -u nftables -f
tail -f /var/log/kern.log | grep nft

# Statistiques de connexions
ss -s
conntrack -S  # Si conntrack installé
```

#### Nettoyage

```bash
# Supprimer les anciens packages iptables (optionnel, après validation)
# ATTENTION : Certaines applications peuvent encore en dépendre
# apt remove iptables  # À faire seulement si vous êtes SÛR

# Nettoyer les anciens scripts
find /etc -name "*iptables*" -type f
# Évaluer et supprimer/adapter chaque script trouvé
```

---

## Cohabitation iptables/nftables

### Pourquoi faire cohabiter ?

La cohabitation peut être nécessaire dans certains cas :
- Migration progressive par étapes
- Applications legacy qui gèrent iptables
- Période de transition pour former l'équipe
- Environnements mixtes (plusieurs admins)

> [!warning] Risques de la cohabitation
> - Règles qui se contredisent entre les deux systèmes
> - Difficulté à déboguer (quelle règle s'applique ?)
> - Performance dégradée (double traitement)
> - Confusion dans la maintenance

### Comment faire cohabiter (approche technique)

```bash
# Les deux systèmes utilisent netfilter, mais avec des priorities différentes
# iptables : priority 0 (par défaut)
# nftables : priority personnalisable

# Configuration nftables avec priorité après iptables
nft add table inet filter
nft add chain inet filter input { type filter hook input priority 10 \; policy accept \; }
nft add chain inet filter forward { type filter hook forward priority 10 \; policy accept \; }
nft add chain inet filter output { type filter hook output priority 10 \; policy accept \; }

# Comme iptables a priority 0, ses règles s'appliquent en PREMIER
# Puis les règles nftables (priority 10) s'appliquent ensuite
```

> [!info] Ordre d'évaluation avec cohabitation
> 1. Règles iptables (priority 0 par défaut)
> 2. Règles nftables (priority > 0)
> 
> Si iptables DROP un paquet, nftables ne le verra jamais !

### Stratégies de cohabitation

#### Stratégie 1 : Séparation par fonction

```bash
# iptables gère le NAT (legacy applications)
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# nftables gère le filtrage (nouvelles règles)
nft add table inet filter
nft add chain inet filter input { type filter hook input priority 0 \; policy drop \; }
nft add rule inet filter input ct state established,related accept
```

#### Stratégie 2 : Séparation par interface

```bash
# iptables pour l'interface externe (applications qui le gèrent)
iptables -A INPUT -i eth0 -p tcp --dport 80 -j ACCEPT

# nftables pour l'interface interne (gestion manuelle)
nft add rule inet filter input iif eth1 accept
```

#### Stratégie 3 : Migration progressive par service

```bash
# Semaine 1 : Migrer uniquement SSH
nft add rule inet filter input tcp dport 22 accept
# Désactiver la règle iptables correspondante
iptables -D INPUT -p tcp --dport 22 -j ACCEPT

# Semaine 2 : Migrer HTTP/HTTPS
nft add rule inet filter input tcp dport { 80, 443 } accept
iptables -D INPUT -p tcp --dport 80 -j ACCEPT
iptables -D INPUT -p tcp --dport 443 -j ACCEPT

# ... etc jusqu'à migration complète
```

### Outils pour gérer la cohabitation

```bash
# Lister TOUTES les règles actives (iptables + nftables)
echo "=== IPTABLES ==="
iptables -L -n -v
iptables -t nat -L -n -v
echo "=== NFTABLES ==="
nft list ruleset

# Script pour détecter les conflits potentiels
cat > /tmp/check-conflicts.sh << 'EOF'
#!/bin/bash
echo "Vérification des conflits iptables/nftables..."

# Ports en écoute
PORTS=$(ss -tuln | awk 'NR>1 {print $5}' | cut -d: -f2 | sort -u)

for port in $PORTS; do
    IPT_RULES=$(iptables -L -n | grep -c "dpt:$port")
    NFT_RULES=$(nft list ruleset | grep -c "dport $port")
    
    if [ $IPT_RULES -gt 0 ] && [ $NFT_RULES -gt 0 ]; then
        echo "⚠️  CONFLIT potentiel sur le port $port"
        echo "   - iptables: $IPT_RULES règles"
        echo "   - nftables: $NFT_RULES règles"
    fi
done
EOF

chmod +x /tmp/check-conflicts.sh
/tmp/check-conflicts.sh
```

> [!tip] Recommandation forte
> La cohabitation doit être **temporaire** (quelques semaines maximum). Planifiez une migration complète dès que possible pour éviter les problèmes à long terme.

---

## Exemples de conversion

### Exemple 1 : Firewall simple de serveur web

#### Configuration iptables d'origine

```bash
# Politique par défaut
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Loopback
iptables -A INPUT -i lo -j ACCEPT

# États établis
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# ICMP
iptables -A INPUT -p icmp -j ACCEPT

# SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# HTTP/HTTPS
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Log et drop
iptables -A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables-DROP: "
iptables -A INPUT -j DROP
```

#### Conversion automatique

```bash
# Via iptables-translate (produit du code non optimal)
nft add table ip filter
nft add chain ip filter INPUT { type filter hook input priority 0 \; policy drop \; }
nft add chain ip filter FORWARD { type filter hook forward priority 0 \; policy drop \; }
nft add chain ip filter OUTPUT { type filter hook output priority 0 \; policy accept \; }
nft add rule ip filter INPUT iif "lo" counter accept
nft add rule ip filter INPUT ct state established,related counter accept
nft add rule ip filter INPUT ip protocol icmp counter accept
nft add rule ip filter INPUT tcp dport 22 counter accept
nft add rule ip filter INPUT tcp dport 80 counter accept
nft add rule ip filter INPUT tcp dport 443 counter accept
nft add rule ip filter INPUT limit rate 5/minute log prefix "iptables-DROP: "
nft add rule ip filter INPUT counter drop
```

#### Conversion optimisée manuelle

```bash
#!/usr/sbin/nft -f

# Vider la configuration
flush ruleset

# Table unifiée IPv4/IPv6
table inet filter {
    # Chaîne INPUT
    chain input {
        type filter hook input priority filter; policy drop;
        
        # Loopback
        iif lo accept
        
        # États établis (règle prioritaire pour performance)
        ct state established,related accept
        ct state invalid drop
        
        # ICMP (IPv4 et IPv6)
        ip protocol icmp accept
        ip6 nexthdr icmpv6 accept
        
        # Services (utilisation d'un set anonyme)
        tcp dport { 22, 80, 443 } accept
        
        # Log et drop avec rate limit
        limit rate 5/minute log prefix "nft-DROP: "
        drop
    }
    
    # Chaînes FORWARD et OUTPUT
    chain forward {
        type filter hook forward priority filter; policy drop;
    }
    
    chain output {
        type filter hook output priority filter; policy accept;
    }
}
```

**Améliorations apportées :**
- ✅ Famille `inet` pour IPv4+IPv6 simultané
- ✅ Set anonyme pour les ports (plus concis, même performance)
- ✅ ICMP IPv6 ajouté
- ✅ Invalidation des paquets invalides
- ✅ Suppression des `counter` (ajoutés automatiquement)
- ✅ Commentaires et structure claire

### Exemple 2 : Routeur NAT avec DMZ

#### Configuration iptables d'origine

```bash
# Enable forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward

# Filter table
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# INPUT rules
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -i eth1 -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -j DROP

# FORWARD rules
iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT  # LAN → Internet
iptables -A FORWARD -i eth0 -o eth2 -p tcp --dport 80 -j ACCEPT  # Internet → DMZ web
iptables -A FORWARD -i eth0 -o eth2 -p tcp --dport 443 -j ACCEPT
iptables -A FORWARD -j DROP

# NAT table
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to-destination 192.168.2.10
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 -j DNAT --to-destination 192.168.2.10
```

#### Conversion optimisée nftables

```bash
#!/usr/sbin/nft -f

flush ruleset

# Table de filtrage
table inet filter {
    chain input {
        type filter hook input priority filter; policy drop;
        
        iif lo accept
        ct state established,related accept
        ct state invalid drop
        
        # SSH depuis LAN uniquement
        iif eth1 tcp dport 22 accept
        
        drop
    }
    
    chain forward {
        type filter hook forward priority filter; policy drop;
        
        # États établis
        ct state established,related accept
        ct state invalid drop
        
        # LAN → Internet
        iif eth1 oif eth0 accept
        
        # Internet → DMZ (après DNAT)
        iif eth0 oif eth2 tcp dport { 80, 443 } ct status dnat accept
        
        drop
    }
    
    chain output {
        type filter hook output priority filter; policy accept;
    }
}

# Table NAT
table inet nat {
    # Set pour les ports web (préparation pour évolution future)
    set web_ports {
        type inet_service
        elements = { 80, 443 }
    }
    
    chain prerouting {
        type nat hook prerouting priority dstnat; policy accept;
        
        # DNAT vers serveur web DMZ
        iif eth0 tcp dport @web_ports dnat to 192.168.2.10
    }
    
    chain postrouting {
        type nat hook postrouting priority srcnat; policy accept;
        
        # Masquerading LAN → Internet
        oif eth0 masquerade
    }
}
```

**Améliorations :**
- ✅ Structure unifiée filter + nat
- ✅ Set pour les ports web (facilite l'ajout de ports)
- ✅ `ct status dnat` au lieu de règles répétitives
- ✅ Invalidation explicite des paquets invalides
- ✅ Configuration atomique (un seul fichier)

### Exemple 3 : Serveur avec whitelist complexe

#### Configuration iptables d'origine

```bash
# Whitelist pour SSH
iptables -A INPUT -p tcp --dport 22 -s 192.168.1.10 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -s 192.168.1.11 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -s 192.168.1.12 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -s 10.0.0.0/8 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j DROP

# Whitelist pour API (port 8080)
iptables -A INPUT -p tcp --dport 8080 -s 172.16.1.10 -j ACCEPT
iptables -A INPUT -p tcp --dport 8080 -s 172.16.1.11 -j ACCEPT
iptables -A INPUT -p tcp --dport 8080 -s 172.16.1.12 -j ACCEPT
iptables -A INPUT -p tcp --dport 8080 -j DROP
```

#### Conversion optimisée avec sets

```bash
#!/usr/sbin/nft -f

flush ruleset

table inet filter {
    # Sets de whitelists
    set ssh_allowed {
        type ipv4_addr
        flags interval
        elements = { 
            192.168.1.10, 
            192.168.1.11, 
            192.168.1.12,
            10.0.0.0/8
        }
    }
    
    set api_allowed {
        type ipv4_addr
        elements = { 
            172.16.1.10, 
            172.16.1.11, 
            172.16.1.12
        }
    }
    
    chain input {
        type filter hook input priority filter; policy drop;
        
        # Règles de base
        iif lo accept
        ct state established,related accept
        ct state invalid drop
        
        # SSH avec whitelist
        tcp dport 22 ip saddr @ssh_allowed accept
        tcp dport 22 drop
        
        # API avec whitelist
        tcp dport 8080 ip saddr @api_allowed accept
        tcp dport 8080 drop
        
        # Autres règles...
        drop
    }
}
```

**Avantages de la version nftables :**
- ✅ **12 règles iptables** → **4 règles nftables + 2 sets**
- ✅ Ajout/suppression d'IP dynamique : `nft add element inet filter ssh_allowed { 192.168.1.13 }`
- ✅ Flag `interval` permet d'ajouter des CIDR après création
- ✅ Performance : lookup en O(1) vs O(n) pour les règles linéaires
- ✅ Lisibilité : intention claire avec des sets nommés

### Exemple 4 : Rate limiting et protection anti-DDoS

#### Configuration iptables d'origine

```bash
# Limite de connexions SSH
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set --name SSH
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 60 --hitcount 4 --rttl --name SSH -j DROP

# Limite de requêtes HTTP
iptables -A INPUT -p tcp --dport 80 -m state --state NEW -m limit --limit 100/sec --limit-burst 200 -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -m state --state NEW -j DROP

# Protection SYN flood
iptables -A INPUT -p tcp --syn -m limit --limit 1/s --limit-burst 3 -j ACCEPT
iptables -A INPUT -p tcp --syn -j DROP
```

#### Conversion nftables

```bash
#!/usr/sbin/nft -f

flush ruleset

table inet filter {
    # Set pour bloquer temporairement les IPs abusives
    set ssh_abusers {
        type ipv4_addr
        flags timeout
    }
    
    chain input {
        type filter hook input priority filter; policy drop;
        
        iif lo accept
        ct state established,related accept
        ct state invalid drop
        
        # Protection SSH (rate limiting par IP source)
        tcp dport 22 ct state new \
            limit rate over 3/minute \
            add @ssh_abusers { ip saddr timeout 1h } \
            drop
        tcp dport 22 ip saddr @ssh_abusers drop
        tcp dport 22 accept
        
        # Protection HTTP (rate limiting global)
        tcp dport 80 ct state new \
            limit rate 100/second burst 200 packets accept
        tcp dport 80 drop
        
        # Protection SYN flood
        tcp flags syn tcp flags != ack \
            limit rate 1/second burst 3 packets accept
        tcp flags syn tcp flags != ack drop
        
        # Autres services...
        tcp dport 443 accept
        
        drop
    }
}
```

**Améliorations :**
- ✅ Set avec timeout automatique pour bannissement temporaire
- ✅ Rate limiting plus expressif et lisible
- ✅ Combinaison de règles (limit + action) sur une ligne
- ✅ Pas besoin du module `recent` (plus simple)

---

## 🎯 Points clés à retenir

1. **Outils de conversion** : `iptables-translate` pour comprendre, conversion manuelle pour optimiser
2. **Méthodologie** : Préparer → Convertir → Tester → Déployer → Valider
3. **Tests obligatoires** : Toujours tester en environnement contrôlé avant la prod
4. **Rollback préparé** : Script de retour arrière prêt avant migration
5. **Cohabitation temporaire** : Possible mais à éviter sur le long terme
6. **Opportunités d'optimisation** : Sets, maps, famille inet, consolidation IPv4/IPv6
7. **Accès console** : Gardez un accès hors-bande en cas de problème
8. **Documentation** : Sauvegarder et documenter avant/pendant/après

> [!warning] Erreurs fréquentes de migration
> - ❌ Migrer en production sans test
> - ❌ Oublier de sauvegarder la config iptables
> - ❌ Ne pas préparer de rollback
> - ❌ Convertir littéralement sans optimiser
> - ❌ Ignorer les applications qui gèrent iptables
> - ❌ Faire la migration un vendredi soir 😅
> - ❌ Ne pas tester tous les flux réseau critiques

> [!tip] Checklist de migration réussie
> - ✅ Backup complet de la config actuelle
> - ✅ Tests en environnement de dev/staging
> - ✅ Script de rollback testé
> - ✅ Documentation de la nouvelle config
> - ✅ Fenêtre de maintenance planifiée
> - ✅ Équipe disponible pendant la migration
> - ✅ Accès console/IPMI garanti
> - ✅ Plan de communication vers les utilisateurs
> - ✅ Monitoring actif pendant et après migration
> - ✅ Formation de l'équipe à nftables

---

*Ce document fait partie de la série sur Netfilter - Partie 5 : Introduction à nftables*