# 📊 Cours : Supervision & Monitoring — Niveau TSSR

> **Objectif** : Comprendre les enjeux de la supervision réseau et système, maîtriser les outils et protocoles utilisés en entreprise, et savoir mettre en place une stratégie de monitoring efficace.

---

## PARTIE 1 — Les bases de la supervision

---

## 1. Pourquoi superviser ?

Sans supervision, tu découvres les pannes quand les utilisateurs appellent. Avec une supervision bien configurée, tu les détectes **avant** qu'elles impactent les utilisateurs — ou au minimum, tu sais exactement ce qui s'est passé et à quel moment.

### Les trois objectifs de la supervision

|Objectif|Description|Exemple|
|---|---|---|
|**Disponibilité**|Savoir si un équipement ou service est en ligne|Alerte si le serveur web ne répond plus|
|**Performance**|Surveiller les métriques dans le temps|CPU à 95% depuis 10 minutes|
|**Capacité**|Anticiper les saturation avant qu'elles arrivent|Disque à 80% → va être plein dans 3 jours|

### Les deux types de supervision

|Type|Description|Exemple|
|---|---|---|
|**Active**|Le système de supervision **interroge** régulièrement les équipements|Ping toutes les 5 minutes, requête SNMP|
|**Passive**|Les équipements **envoient** eux-mêmes leurs informations|Syslog, SNMP Trap, métriques push|

> 💡 En pratique, on combine les deux : supervision active pour détecter les pannes, passive pour recevoir les alertes en temps réel.

---

## 2. Les métriques essentielles à surveiller

### Sur les serveurs

|Métrique|Seuil d'alerte courant|Seuil critique|
|---|---|---|
|CPU|> 80% pendant 5 min|> 95%|
|RAM|> 85%|> 95%|
|Swap|> 50%|> 80%|
|Disque|> 80%|> 90%|
|Charge système (load)|> nb de cœurs|> 2× nb de cœurs|
|Température CPU|> 70°C|> 85°C|

### Sur le réseau

|Métrique|Ce qu'on surveille|
|---|---|
|Disponibilité (uptime)|Le lien est-il actif ?|
|Bande passante|Taux d'utilisation du lien (%)|
|Latence|Temps de réponse (ms)|
|Perte de paquets|% de paquets perdus|
|Erreurs d'interface|CRC, collisions, discards|
|Nombre de connexions|Sessions TCP actives|

### Sur les services applicatifs

|Service|Métrique clé|
|---|---|
|Serveur web|Temps de réponse HTTP, taux d'erreurs (5xx)|
|Base de données|Temps de requête, connexions actives, deadlocks|
|DNS|Temps de résolution, taux d'échec|
|Messagerie|File d'attente SMTP, taux de bounce|

---

## PARTIE 2 — Le protocole SNMP

---

## 3. Qu'est-ce que SNMP ?

**SNMP = Simple Network Management Protocol**

C'est le protocole standard pour superviser les équipements réseau et serveurs. Il permet de **collecter des informations** et d'**envoyer des alertes**.

### Les versions SNMP

|Version|Sécurité|Usage|
|---|---|---|
|**SNMPv1**|Aucune (community string en clair)|❌ Obsolète|
|**SNMPv2c**|Community string en clair|⚠️ Encore très répandu mais non chiffré|
|**SNMPv3**|Authentification + chiffrement|✅ Recommandé en production|

> ⚠️ SNMPv1 et v2c utilisent une "community string" (comme un mot de passe) transmise **en clair** sur le réseau. Toujours utiliser SNMPv3 pour les nouvelles installations.

---

## 4. Le fonctionnement de SNMP

### Les composants

```
┌─────────────────────────────────────────────────────┐
│                  Système de supervision              │
│                  (Nagios, Zabbix...)                │
│                   SNMP Manager                       │
└──────────────────────┬──────────────────────────────┘
                       │ Requêtes SNMP (UDP 161)
                       │ ←── SNMP Traps (UDP 162)
         ┌─────────────┼─────────────┐
         ▼             ▼             ▼
   [Routeur]     [Serveur Linux]   [Switch]
   SNMP Agent    SNMP Agent        SNMP Agent
```

### Les opérations SNMP

|Opération|Direction|Description|
|---|---|---|
|**GET**|Manager → Agent|Récupérer la valeur d'un objet|
|**GETNEXT**|Manager → Agent|Récupérer l'objet suivant|
|**GETBULK**|Manager → Agent|Récupérer plusieurs objets en une fois (v2+)|
|**SET**|Manager → Agent|Modifier une valeur sur l'agent|
|**TRAP**|Agent → Manager|L'agent envoie une alerte de sa propre initiative|
|**INFORM**|Agent → Manager|Comme TRAP mais avec accusé de réception (v2+)|

> 💡 **La TRAP est essentielle** : c'est ce qui permet à un équipement de signaler immédiatement un problème (lien down, température critique) sans attendre la prochaine interrogation du manager.

---

## 5. La MIB — l'annuaire des objets SNMP

**MIB = Management Information Base**

C'est une base de données hiérarchique qui définit tous les objets qu'un équipement peut exposer via SNMP. Chaque objet est identifié par un **OID (Object Identifier)**.

```
Exemple d'OID : 1.3.6.1.2.1.1.1.0
                │ │ │ │ │ │ │ │ └── Instance .0
                │ │ │ │ │ │ │ └──── sysDescr (description du système)
                │ │ │ │ │ │ └────── system (infos système)
                │ │ │ │ │ └──────── mib-2
                │ │ │ │ └────────── mgmt
                │ │ │ └──────────── internet
                │ │ └────────────── dod
                │ └──────────────── org
                └────────────────── iso
```

### OIDs courants à connaître

|OID|Description|
|---|---|
|`1.3.6.1.2.1.1.1.0`|sysDescr — Description du système|
|`1.3.6.1.2.1.1.3.0`|sysUpTime — Temps de fonctionnement|
|`1.3.6.1.2.1.1.5.0`|sysName — Nom de l'équipement|
|`1.3.6.1.2.1.2.2`|ifTable — Table des interfaces réseau|
|`1.3.6.1.2.1.25.3.3.1.2`|hrProcessorLoad — Charge CPU|

```bash
# Interroger un équipement SNMP en ligne de commande
snmpget -v2c -c public 192.168.1.1 1.3.6.1.2.1.1.1.0
snmpwalk -v2c -c public 192.168.1.1 1.3.6.1.2.1.1
snmpget -v3 -u admin -l authPriv -a SHA -A "monMDP" -x AES -X "maClé" 192.168.1.1 sysUpTime.0
```

---

## PARTIE 3 — Syslog

---

## 6. Le protocole Syslog

**Syslog** est le protocole standard pour la **centralisation des logs** sur le réseau. Chaque équipement envoie ses logs à un serveur central (syslog server).

- Port : **UDP 514** (traditionnel) ou **TCP 514** (plus fiable) ou **TCP 6514** (chiffré TLS)
- Protocole standard : RFC 5424

### Les niveaux de sévérité Syslog

|Niveau|Numéro|Description|Exemple|
|---|---|---|---|
|**Emergency**|0|Système inutilisable|Kernel panic|
|**Alert**|1|Action immédiate requise|Perte de redondance critique|
|**Critical**|2|Condition critique|Défaillance matérielle|
|**Error**|3|Erreur|Service qui plante|
|**Warning**|4|Avertissement|Disque à 85%|
|**Notice**|5|Événement notable normal|Connexion SSH réussie|
|**Informational**|6|Information|Démarrage d'un service|
|**Debug**|7|Débogage|Très verbeux, pour le développement|

> 💡 **Moyen mémo-technique** : **E**very **A**dmin **C**an **E**arn **W**orkplace **N**ods **I**f **D**iligent → Emergency, Alert, Critical, Error, Warning, Notice, Informational, Debug

### Les facilités Syslog (facility)

La facility identifie **qui génère le message** :

|Facility|Source|
|---|---|
|kern|Noyau Linux|
|auth / authpriv|Authentification|
|daemon|Services systèmes|
|cron|Tâches planifiées|
|mail|Messagerie|
|local0 à local7|Utilisation personnalisée (équipements réseau)|

---

## 7. Centralisation des logs — architecture

```
[Routeurs]    ──────────────────────────┐
[Switches]    ──────── Syslog UDP/514 ──┤
[Serveurs]    ──────────────────────────┤──→ [Serveur Syslog central]
[Firewalls]   ──────────────────────────┘         (rsyslog, syslog-ng)
                                                         │
                                              ┌──────────┘
                                              ▼
                                    [SIEM / Analyse]
                                    (Graylog, ELK, Splunk)
```

### Pourquoi centraliser les logs ?

- Un équipement compromis peut **effacer ses logs locaux** → les logs centralisés restent intacts
- **Corrélation** : relier les événements de plusieurs équipements pour détecter une attaque
- **Conformité** : RGPD, ISO 27001, PCI-DSS exigent souvent la conservation des logs
- **Recherche rapide** : un seul endroit pour chercher dans tous les logs

---

## PARTIE 4 — Les outils de supervision

---

## 8. Nagios / Nagios Core

**Nagios** est l'un des outils de supervision les plus répandus. Il fonctionne sur le principe de **checks actifs** : il interroge régulièrement les hôtes et services.

### Concepts clés

|Concept|Description|
|---|---|
|**Host**|Un équipement à surveiller (serveur, routeur...)|
|**Service**|Un check sur un hôte (ping, port HTTP, CPU...)|
|**Check**|Test exécuté pour vérifier l'état d'un service|
|**Plugin**|Script qui effectue le check (check_ping, check_http...)|
|**Contact**|Personne qui reçoit les alertes|
|**Notification**|Email/SMS envoyé lors d'un changement d'état|

### Les états Nagios

|État|Couleur|Signification|
|---|---|---|
|**OK**|🟢 Vert|Tout va bien|
|**WARNING**|🟡 Jaune|Seuil d'avertissement dépassé|
|**CRITICAL**|🔴 Rouge|Seuil critique dépassé|
|**UNKNOWN**|⚪ Gris|Impossible de déterminer l'état|

### Plugins Nagios courants

```bash
check_ping -H 192.168.1.1                              # Ping
check_http -H www.site.com -p 80                       # HTTP
check_tcp -H 192.168.1.1 -p 443                        # Port TCP
check_disk -w 80% -c 90% -p /                          # Disque
check_cpu -w 80 -c 95                                  # CPU
check_mem -w 85 -c 95                                  # RAM
check_snmp -H 192.168.1.1 -o sysUpTime.0              # SNMP
```

---

## 9. Zabbix

**Zabbix** est une solution de supervision plus moderne et complète que Nagios. Il offre une interface graphique riche, des graphiques intégrés et une gestion avancée des alertes.

### Architecture Zabbix

```
[Équipements supervisés]
        │
        │ Zabbix Agent (actif ou passif)
        │ SNMP
        │ IPMI
        ▼
[Zabbix Server] ──→ [Base de données MySQL/PostgreSQL]
        │
        ▼
[Interface Web Zabbix]
```

### Concepts clés Zabbix

|Concept|Description|
|---|---|
|**Item**|Une métrique collectée (CPU, RAM, débit...)|
|**Trigger**|Condition qui déclenche une alerte (CPU > 90%)|
|**Action**|Ce qui se passe quand un trigger se déclenche (email, script)|
|**Template**|Ensemble d'items et triggers réutilisables|
|**Host group**|Regroupement logique d'hôtes|
|**Map**|Vue cartographique du réseau|

### Modes de collecte Zabbix

|Mode|Description|
|---|---|
|**Agent actif**|L'agent envoie les métriques au serveur Zabbix|
|**Agent passif**|Le serveur Zabbix interroge l'agent|
|**SNMP**|Collecte via SNMP (équipements sans agent)|
|**IPMI**|Supervision matérielle (serveurs physiques)|
|**JMX**|Supervision Java|
|**Sans agent**|Ping, TCP check, HTTP check|

---

## 10. Autres outils à connaître

|Outil|Type|Usage|
|---|---|---|
|**Prometheus + Grafana**|Métriques + visualisation|Environnements cloud/containers, très populaire|
|**ELK Stack** (Elasticsearch, Logstash, Kibana)|Logs|Centralisation et analyse de logs à grande échelle|
|**Graylog**|Logs|Alternative à ELK, plus simple à administrer|
|**PRTG**|Supervision complète|Solution Windows très utilisée en PME|
|**Centreon**|Supervision|Fork de Nagios, très populaire en France|
|**Datadog**|SaaS|Supervision cloud, coûteux mais puissant|
|**Netflow / sFlow**|Trafic réseau|Analyse des flux réseau (qui parle à qui, combien)|
|**LibreNMS**|Réseau|Supervision réseau open-source, autodiscovery|

---

## PARTIE 5 — Sauvegardes et PRA/PCA

---

## 11. Les stratégies de sauvegarde

### Les types de sauvegarde

|Type|Description|Avantages|Inconvénients|
|---|---|---|---|
|**Complète**|Sauvegarde de toutes les données|Simple à restaurer|Longue, volumineuse|
|**Différentielle**|Tout ce qui a changé depuis la dernière **complète**|Restauration rapide (complète + 1 diff)|Grossit au fil du temps|
|**Incrémentale**|Tout ce qui a changé depuis la dernière **sauvegarde** (quelle qu'elle soit)|Rapide, peu volumineuse|Restauration longue (complète + toutes les incrémentales)|

### La règle 3-2-1

```
3 copies des données
  │
  ├── 2 supports différents (ex: disque local + NAS)
  │
  └── 1 copie hors site (cloud, autre bâtiment, bande externalisée)
```

> 💡 **La règle 3-2-1 est le minimum acceptable**. Un ransomware qui chiffre le serveur ET le NAS local en même temps est un scénario réel. La copie hors site sauve la mise.

### Fréquence des sauvegardes

|Donnée|Fréquence recommandée|
|---|---|
|Bases de données critiques|Toutes les heures ou en continu (log shipping)|
|Données applicatives|Quotidienne|
|Configurations système|À chaque modification + quotidienne|
|Images de VM|Hebdomadaire + avant toute modification|

---

## 12. PRA et PCA — les plans de continuité

### PCA — Plan de Continuité d'Activité

Le PCA définit comment **maintenir l'activité** pendant une crise. L'objectif est de ne jamais s'arrêter.

### PRA — Plan de Reprise d'Activité

Le PRA définit comment **reprendre l'activité** après une interruption. L'objectif est de redémarrer le plus vite possible.

### Les indicateurs clés

|Indicateur|Définition|Exemple|
|---|---|---|
|**RTO** (Recovery Time Objective)|Durée maximale acceptable d'interruption|"On peut se permettre 4h d'arrêt maximum"|
|**RPO** (Recovery Point Objective)|Perte de données maximale acceptable|"On peut perdre au maximum 1h de données"|

```
Incident                           Reprise
    │                                 │
    ▼                                 ▼
────┼─────────────────────────────────┼────
    │◄──────────── RTO ───────────────►│
    │
    │◄── RPO ──►│
Dernière     Incident
sauvegarde
```

> 💡 **RTO et RPO déterminent tes choix techniques** :
> 
> - RPO = 0 → réplication synchrone en temps réel (coûteux)
> - RPO = 1h → sauvegardes horaires
> - RTO = 15 min → clustering, failover automatique
> - RTO = 4h → restauration manuelle acceptable

### Les niveaux de tolérance

|Niveau|RTO|RPO|Solution technique|
|---|---|---|---|
|Critique|< 15 min|< 5 min|Clustering actif/actif, réplication synchrone|
|Haute dispo|< 1h|< 30 min|Failover automatique, réplication asynchrone|
|Standard|< 4h|< 1h|Sauvegarde horaire + restauration manuelle|
|Basique|< 24h|< 24h|Sauvegarde quotidienne|

---

## PARTIE 6 — Scénarios de dépannage

---

### 🔴 Scénario 1 — Un hôte disparaît de la supervision

**Situation** :

- Nagios/Zabbix remonte une alerte : `serveur-bdd01` est en état CRITICAL / DOWN
- Les utilisateurs de l'application commencent à signaler des erreurs
- L'alerte est arrivée à 14h32

**Questions à se poser** :

1. L'hôte répond-il au ping depuis le serveur de supervision ?
2. Le problème est-il réseau ou système ?
3. D'autres équipements du même réseau sont-ils impactés ?
4. Y a-t-il eu un changement récent (mise à jour, intervention) ?

**Marche à suivre** :

```bash
# 1. Depuis le serveur de supervision
ping 192.168.1.100                      # Le serveur répond-il ?
traceroute 192.168.1.100                # Où s'arrête le trafic ?

# 2. Si le ping fonctionne mais l'agent/service ne répond plus
telnet 192.168.1.100 10050              # Port de l'agent Zabbix
telnet 192.168.1.100 5666               # Port NRPE (Nagios)

# 3. Sur le serveur lui-même (si accès console)
systemctl status zabbix-agent           # L'agent est-il démarré ?
journalctl -u zabbix-agent -n 50        # Logs de l'agent
```

> 💡 **Premier réflexe** : regarder si d'autres hôtes du même VLAN ou du même switch sont également down. Si oui → problème réseau. Si un seul hôte → problème sur l'hôte lui-même.

---

### 🔴 Scénario 2 — Faux positifs en masse dans la supervision

**Situation** :

- Le serveur de supervision envoie des dizaines d'alertes
- La moitié des hôtes semblent DOWN
- En vérifiant manuellement, les serveurs répondent correctement
- Le problème est apparu d'un coup à 3h du matin

**Questions à se poser** :

1. Le serveur de supervision lui-même a-t-il un problème réseau ?
2. Le serveur de supervision est-il surchargé ?
3. Y a-t-il eu un incident réseau cette nuit (coupure, maintenance) ?
4. Les checks sont-ils mal configurés (timeout trop court) ?

**Analyse** :

Des alertes en masse simultanées pointent rarement vers une vraie panne généralisée. C'est souvent le **serveur de supervision lui-même** ou sa **connectivité réseau** qui est en cause.

**Marche à suivre** :

1. Vérifier l'état du serveur de supervision (CPU, RAM, disque)
2. Vérifier la connectivité réseau du serveur de supervision
3. Regarder si des maintenances réseau ont eu lieu cette nuit
4. Vérifier les logs du serveur de supervision
5. Augmenter le nombre de tentatives avant alerte (check_attempts) pour éviter les faux positifs

> 💡 **Règle de base** : configurer au moins **3 tentatives** avant de passer en état CRITICAL. Une seule tentative échouée peut être un faux positif (timeout réseau momentané).

---

### 🔴 Scénario 3 — Disque d'un serveur qui va saturer

**Situation** :

- Zabbix remonte un warning : `/var` est à 82% sur `serveur-app01`
- Selon la tendance, le disque sera plein dans 48 heures
- Le seuil critique est à 90%

**Questions à se poser** :

1. Quel type de données remplit ce disque ?
2. La croissance est-elle normale (logs) ou anormale (fuite mémoire, dump) ?
3. Peut-on libérer de l'espace rapidement ?
4. Faut-il étendre la partition ?

**Marche à suivre** :

```bash
# Identifier ce qui remplit le disque
df -h                                   # Vue globale
du -sh /var/* | sort -rh | head -10    # Top consommateurs dans /var
du -sh /var/log/* | sort -rh | head -10 # Logs spécifiques

# Libérer de l'espace rapidement
find /var/log -name "*.log" -mtime +30 -delete   # Supprimer les vieux logs
journalctl --vacuum-size=200M           # Réduire les logs systemd
apt clean                               # Nettoyer le cache apt (si Ubuntu/Debian)

# Vérifier logrotate
cat /etc/logrotate.conf
ls /etc/logrotate.d/                    # Configurations par service

# Étendre si nécessaire
lvextend -L +10G /dev/vg0/var           # Étendre le LV (si LVM)
resize2fs /dev/vg0/var                  # Redimensionner le FS
```

---

### 🔴 Scénario 4 — Une alerte de performance CPU récurrente

**Situation** :

- Tous les jours à 2h du matin, une alerte CPU se déclenche sur `serveur-bdd01`
- Le CPU monte à 95% pendant environ 20 minutes, puis redescend
- Pas d'impact utilisateur (c'est la nuit)
- L'alerte se répète depuis 2 semaines

**Questions à se poser** :

1. Y a-t-il une tâche planifiée à 2h du matin ?
2. Quelle est la durée exacte du pic ? Est-elle constante ?
3. S'agit-il d'un problème ou d'un comportement normal (sauvegarde, maintenance) ?
4. Faut-il ajuster les seuils ou la tâche planifiée ?

**Analyse** :

Un pic CPU régulier, à heure fixe, de durée constante → **tâche planifiée** (cron job, sauvegarde, indexation, antivirus...).

**Marche à suivre** :

```bash
# Sur le serveur, voir les tâches planifiées
crontab -l                              # Tâches de l'utilisateur courant
cat /etc/crontab                        # Tâches système
ls /etc/cron.daily/ /etc/cron.weekly/   # Scripts planifiés

# Voir quel processus consomme le CPU à 2h (dans les logs)
grep "02:0" /var/log/syslog | grep -i "start\|begin\|launch"

# Créer une plage de maintenance dans la supervision
# (Nagios/Zabbix : "downtime" / "maintenance period")
# Permet de ne pas recevoir d'alertes pendant les opérations planifiées
```

> 💡 **À faire** : créer une **plage de maintenance** dans Nagios/Zabbix pour couvrir la fenêtre de backup. Les alertes seront supprimées pendant cette période, évitant les faux positifs nocturnes.

---

### 🔴 Scénario 5 — Perte de logs : un incident sans traces

**Situation** :

- Un incident de sécurité a eu lieu sur un serveur
- L'équipe de sécurité veut analyser les logs pour comprendre ce qui s'est passé
- Les logs locaux du serveur ont été partiellement effacés
- Il n'y a pas de serveur syslog centralisé

**Questions à se poser** :

1. Reste-t-il des logs partiels dans `/var/log` ?
2. Y a-t-il des logs dans d'autres équipements (firewall, switch, proxy) ?
3. La supervision a-t-elle des données historiques ?
4. Comment éviter que cela se reproduise ?

**Analyse** :

Sans centralisation des logs, une compromission peut effacer les preuves. C'est exactement pour ça qu'un **serveur syslog centralisé** est indispensable.

**Marche à suivre immédiate** :

```bash
# Chercher des logs résiduels
find /var/log -name "*.gz" -o -name "*.1"   # Archives de logrotate
last                                         # Historique connexions (si /var/log/wtmp intact)
lastb                                        # Connexions échouées
journalctl --since "2024-01-15"              # Logs systemd si non effacés
```

**Mesures correctives à mettre en place** :

1. Installer et configurer **rsyslog** sur tous les serveurs
2. Mettre en place un **serveur syslog central** (rsyslog, syslog-ng, Graylog)
3. Configurer les équipements réseau (firewall, switch) pour envoyer leurs logs au syslog central
4. Définir une politique de rétention des logs (durée légale selon le secteur : souvent 1 an minimum)

---

### 🔴 Scénario 6 — La supervision ne reçoit plus les métriques d'un agent

**Situation** :

- Zabbix indique que `serveur-web02` est "non supporté" pour plusieurs métriques
- Le serveur lui-même est opérationnel (ping OK, services OK)
- L'agent Zabbix tourne sur le serveur
- Le problème a commencé après une mise à jour de l'OS

**Questions à se poser** :

1. La version de l'agent Zabbix est-elle compatible avec le serveur Zabbix ?
2. Le firewall autorise-t-il la communication entre le serveur Zabbix et l'agent ?
3. La configuration de l'agent pointe-t-elle vers le bon serveur Zabbix ?
4. Y a-t-il des erreurs dans les logs de l'agent ?

**Marche à suivre** :

```bash
# Sur le serveur supervisé
systemctl status zabbix-agent2          # L'agent tourne-t-il ?
journalctl -u zabbix-agent2 -n 100      # Logs de l'agent
cat /etc/zabbix/zabbix_agent2.conf | grep -E "Server|ServerActive|Hostname"

# Vérifier la connectivité
telnet <IP_Zabbix_Server> 10051         # Port actif (agent → serveur)
# ou
ss -tuln | grep 10050                   # Port passif (serveur → agent)

# Tester manuellement une métrique
zabbix_agent2 -t system.cpu.util        # Tester localement un item

# Firewall
ufw status | grep 10050
ufw allow from <IP_Zabbix> to any port 10050
```

---

## 13. Exercices d'entraînement

---

**Exercice 1** — Tu dois mettre en place la supervision d'un nouveau site avec 50 serveurs Linux et 10 équipements réseau (switches et routeurs). Quelle stratégie adoptes-tu pour :

- Superviser les serveurs Linux ?
- Superviser les équipements réseau ?
- Centraliser les logs ?

<details> <summary>👁️ Voir la réponse</summary>

**Serveurs Linux** :

- Installer l'agent Zabbix (ou NRPE pour Nagios) sur chaque serveur
- Utiliser un template Linux standard pour les métriques CPU, RAM, disque, charge
- Mode agent passif ou actif selon l'architecture réseau (firewall)

**Équipements réseau** :

- Activer SNMPv3 sur tous les équipements (switches, routeurs)
- Configurer le serveur de supervision pour interroger via SNMP
- Activer les SNMP Traps pour les alertes critiques (lien down, température)
- Utiliser LibreNMS ou Zabbix avec templates SNMP constructeurs

**Centralisation des logs** :

- Configurer rsyslog sur tous les serveurs pour envoyer vers un serveur central
- Configurer les équipements réseau pour envoyer leurs Syslog (facility local7)
- Mettre en place Graylog ou ELK pour la visualisation et la recherche

</details>

---

**Exercice 2** — Quelle différence entre RTO et RPO ? Pour une base de données critique, quels objectifs seraient raisonnables et quelles solutions techniques les respecteraient ?

<details> <summary>👁️ Voir la réponse</summary>

- **RTO** = combien de temps peut-on rester en panne → objectif de **durée de reprise**
- **RPO** = combien de données peut-on perdre → objectif de **perte de données**

Pour une BDD critique :

- RTO = 30 min → Failover automatique (clustering MySQL/MariaDB avec Galera, ou Always On pour SQL Server)
- RPO = 5 min → Réplication asynchrone + sauvegarde toutes les 5 minutes (binlog)

Si RPO = 0 est requis → réplication **synchrone** (les deux nœuds confirment avant de valider une transaction). Plus lent mais aucune perte de données.

</details>

---

**Exercice 3** — Un trigger Zabbix déclenche une alerte dès que le CPU dépasse 80% pendant 1 minute. Tu reçois des dizaines d'alertes chaque jour pour des pics légitimes de courte durée. Comment améliores-tu la configuration ?

<details> <summary>👁️ Voir la réponse</summary>

Plusieurs optimisations possibles :

1. **Augmenter la durée** : déclencher l'alerte seulement si le CPU > 80% pendant 5 minutes consécutives (et non 1 minute) → élimine les pics légitimes courts
    
2. **Ajouter un seuil intermédiaire** :
    
    - WARNING : CPU > 80% pendant 5 min
    - CRITICAL : CPU > 95% pendant 2 min
3. **Créer une plage de maintenance** pour les fenêtres de backup/traitement batch
    
4. **Utiliser la tendance** : alerter si le CPU est élevé ET la charge système est > nombre de cœurs × 2
    
5. **Dépendances** : si un process batch tourne (cron), inhiber l'alerte CPU temporairement
    

</details>

---

## 14. Aide-mémoire rapide

```
SNMP
v1/v2c → community string en clair (⚠️ non sécurisé)
v3     → authentification + chiffrement (✅ recommandé)
UDP 161 → requêtes SNMP (manager vers agent)
UDP 162 → SNMP Traps (agent vers manager)
GET / GETNEXT / GETBULK / SET / TRAP / INFORM

SYSLOG — NIVEAUX (0 à 7, du plus au moins critique)
0 Emergency  1 Alert    2 Critical  3 Error
4 Warning    5 Notice   6 Info      7 Debug
Moyen mnémo : Every Admin Can Earn Workplace Nods If Diligent

SAUVEGARDES
Complète    → tout, simple à restaurer, volumineuse
Différentielle → depuis la dernière complète
Incrémentale  → depuis la dernière sauvegarde (quelle qu'elle soit)
Règle 3-2-1 : 3 copies, 2 supports, 1 hors site

RTO / RPO
RTO → temps max d'interruption acceptable
RPO → perte de données max acceptable
RPO faible → réplication synchrone / sauvegarde fréquente
RTO faible → clustering / failover automatique

NAGIOS — ÉTATS
OK (vert) → WARNING (jaune) → CRITICAL (rouge) → UNKNOWN (gris)

ZABBIX — CONCEPTS
Item    → métrique collectée
Trigger → condition d'alerte (CPU > 90%)
Action  → réaction (email, script)
Template → ensemble réutilisable d'items + triggers

OUTILS
Nagios / Centreon → supervision active, checks
Zabbix / PRTG    → supervision complète, graphes intégrés
Prometheus+Grafana → métriques cloud/containers
ELK / Graylog    → centralisation et analyse de logs
LibreNMS         → supervision réseau, autodiscovery
```

---

> ✅ **À retenir** : La supervision n'est pas une option — c'est l'infrastructure de ton infrastructure. Sans elle, tu es aveugle. Trois règles d'or :
> 
> 1. **Superviser proactivement** : détecter avant les utilisateurs
> 2. **Centraliser les logs** : un log effacé localement n'existe plus
> 3. **Tester les sauvegardes** : une sauvegarde non testée n'est pas une sauvegarde