# journalctl - Cheatsheet TSSR
> CCP 3 — Exploiter des serveurs Linux

## Concepts clés

| Concept | Description |
|--------|-------------|
| journald | Daemon systemd qui collecte et stocke les logs |
| Journal binaire | Logs stockés en format binaire — lus via `journalctl` |
| Persistance | Par défaut volatile (`/run/log/journal`) — persistant si `/var/log/journal/` existe |
| Boot ID | Identifiant unique de chaque démarrage système |
| Priorité | Niveaux de sévérité : emerg(0) alert(1) crit(2) err(3) warning(4) notice(5) info(6) debug(7) |

## Filtres essentiels

| Action | Commande |
|--------|----------|
| Tous les logs | `journalctl` |
| Logs en temps réel | `journalctl -f` |
| Logs d'un service | `journalctl -u nginx` |
| Logs d'un service en temps réel | `journalctl -fu nginx` |
| Logs du boot actuel | `journalctl -b` |
| Logs du boot précédent | `journalctl -b -1` |
| Lister les boots disponibles | `journalctl --list-boots` |
| Filtrer par priorité (erreurs et +) | `journalctl -p err` |
| Filtrer par priorité (warnings et +) | `journalctl -p warning` |
| Filtrer depuis une date | `journalctl --since "2024-01-15 08:00:00"` |
| Filtrer sur une plage | `journalctl --since "1 hour ago" --until "now"` |
| Logs d'un PID | `journalctl _PID=1234` |
| Logs d'un utilisateur | `journalctl _UID=1000` |
| Dernières N lignes | `journalctl -n 50` |

## Formats de sortie

| Action | Commande |
|--------|----------|
| Format court (défaut) | `journalctl -o short` |
| Format verbeux | `journalctl -o verbose` |
| Format JSON lisible | `journalctl -o json-pretty` |
| Sans pagination (pipe-friendly) | `journalctl --no-pager` |
| Combiner avec grep | `journalctl -u ssh --no-pager \| grep "Failed"` |

## Gestion du journal

| Action | Commande |
|--------|----------|
| Taille occupée sur disque | `journalctl --disk-usage` |
| Purger logs > 7 jours | `journalctl --vacuum-time=7d` |
| Limiter à 500 Mo | `journalctl --vacuum-size=500M` |
| Activer la persistance | `mkdir -p /var/log/journal && systemctl restart systemd-journald` |

## Fichiers importants

| Fichier / Dossier | Contenu |
|-------------------|---------|
| `/etc/systemd/journald.conf` | Configuration de journald (rétention, taille, persistance) |
| `/var/log/journal/` | Logs persistants (si activé) |
| `/run/log/journal/` | Logs volatils (effacés au reboot) |

## Points de vigilance

| Risque / Erreur | Explication |
|-----------------|-------------|
| Logs perdus au reboot | Sans persistance, le journal est effacé à chaque démarrage |
| `journalctl` sans filtre | Affiche tout depuis l'origine — toujours filtrer en prod |
| `-p err` inclut le dessus | `err` affiche aussi `crit`, `alert`, `emerg` |
| Pagination par défaut | Utiliser `--no-pager` pour rediriger vers grep ou un fichier |
| Rotation automatique | Configurée dans `journald.conf` — vérifier `SystemMaxUse=` |
