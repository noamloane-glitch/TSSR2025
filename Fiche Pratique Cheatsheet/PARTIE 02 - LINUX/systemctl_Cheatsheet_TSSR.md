# systemctl - Cheatsheet TSSR
> CCP 3 — Exploiter des serveurs Linux

## Concepts clés

| Concept | Description |
|--------|-------------|
| systemd | Gestionnaire de services et d'init (PID 1) sur les distros modernes |
| Unit | Unité de configuration systemd (service, socket, timer, mount…) |
| `.service` | Type d'unit le plus courant — définit un service |
| `active (running)` | Service en cours d'exécution |
| `inactive (dead)` | Service arrêté |
| `failed` | Service en erreur |
| `enabled` | Démarrage automatique au boot activé |
| `disabled` | Démarrage automatique désactivé |
| `masked` | Service bloqué — ne peut pas être démarré |

## Gestion des services

| Action | Commande |
|--------|----------|
| Démarrer un service | `systemctl start nginx` |
| Arrêter un service | `systemctl stop nginx` |
| Redémarrer un service | `systemctl restart nginx` |
| Recharger la config sans redémarrer | `systemctl reload nginx` |
| Statut d'un service | `systemctl status nginx` |
| Activer au démarrage | `systemctl enable nginx` |
| Désactiver au démarrage | `systemctl disable nginx` |
| Activer + démarrer immédiatement | `systemctl enable --now nginx` |
| Désactiver + arrêter immédiatement | `systemctl disable --now nginx` |
| Bloquer un service | `systemctl mask nginx` |
| Débloquer un service | `systemctl unmask nginx` |

## État du système

| Action | Commande |
|--------|----------|
| Lister tous les services actifs | `systemctl list-units --type=service` |
| Lister les services en erreur | `systemctl --failed` |
| Lister tous les services (actifs + inactifs) | `systemctl list-units --type=service --all` |
| Lister les units au démarrage | `systemctl list-unit-files --type=service` |
| Recharger la config systemd | `systemctl daemon-reload` |
| Redémarrer le système | `systemctl reboot` |
| Éteindre le système | `systemctl poweroff` |

## Fichiers importants

| Fichier / Dossier | Contenu |
|-------------------|---------|
| `/etc/systemd/system/` | Units personnalisées ou modifiées (prioritaire) |
| `/lib/systemd/system/` | Units installées par les paquets |
| `/etc/systemd/system/nginx.service.d/` | Overrides partiels (drop-in) |
| `systemctl edit nginx` | Crée un override sans modifier le fichier d'origine |
| `systemctl cat nginx` | Affiche le contenu complet de l'unit active |

## Points de vigilance

| Risque / Erreur | Explication |
|-----------------|-------------|
| Oublier `daemon-reload` | Après toute modification de fichier `.service`, obligatoire |
| `reload` ≠ `restart` | `reload` relit la config sans couper le service (si supporté) |
| `mask` bloque tout | Même `start` manuel est impossible sur un service masqué |
| `enable` sans `start` | Le service ne démarre pas immédiatement — utiliser `--now` |
| Logs associés | `journalctl -u nginx -f` pour suivre les logs en temps réel |
