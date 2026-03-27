# GPO Drive-Mount (E: et F:) - Cheatsheet TSSR
> CCP 2 — Exploiter des serveurs Windows & Active Directory

## Concepts clés

| Concept | Valeur |
|--------|-------------------|
| GPO | Objet de stratégie de groupe appliqué aux OU |
| Drive Maps | Préférence GPO pour monter des lecteurs réseau |
| Chemin UNC | `\\serveur\partage` — format obligatoire |
| Lettre E: | Lecteur réseau mappé (ex. : données communes) |
| Lettre F: | Lecteur réseau mappé (ex. : données utilisateur) |
| Scope | GPO liée à une OU contenant les objets **ordinateurs** ou **utilisateurs** |
| Action | `Create` = crée si absent / `Update` = force la mise à jour |

## Étapes de création

| Étape | Action |
|--------|-------------------|
| 1. Ouvrir la console | `gpmc.msc` |
| 2. Créer la GPO | Clic droit sur l'OU → *Créer et lier une GPO* → nommer `Drive-Mount` |
| 3. Modifier la GPO | Clic droit → *Modifier* |
| 4. Naviguer vers Drive Maps | `Configuration utilisateur > Préférences > Paramètres Windows > Mappages de lecteurs` |
| 5. Créer un lecteur E: | Clic droit → *Nouveau > Lecteur mappé* |
| 6. Renseigner le chemin | `\\serveur\partage_E` — Lettre : `E` — Action : `Update` |
| 7. Créer un lecteur F: | Répéter pour `\\serveur\partage_F` — Lettre : `F` |
| 8. Lier la GPO | GPO déjà liée à l'étape 2, vérifier le scope |
| 9. Forcer l'application | `gpupdate /force` sur un client pour tester |

## Ciblage et filtrage

| Option | Valeur / Usage |
|--------|-------------------|
| Ciblage par groupe | Onglet *Commun* → *Ciblage au niveau des éléments* → Groupe de sécurité |
| Reconnexion | Cocher *Reconnecter* pour que le lecteur persiste après déconnexion |
| Masquer/Afficher | Option *Afficher ce lecteur* / *Afficher tous les lecteurs* |
| Filtrage sécurité GPO | Ajouter le groupe cible dans *Filtrage de sécurité* (enlever `Utilisateurs authentifiés` si besoin) |
| Portée utilisateur | GPO appliquée à la session utilisateur — vérifier que l'OU contient bien les **utilisateurs** |

## Vérification et diagnostic

| Action | Commande / Outil |
|--------|-------------------|
| Forcer l'application GPO | `gpupdate /force` |
| Vérifier les GPO appliquées | `gpresult /r` ou `gpresult /h rapport.html` |
| Lister les lecteurs montés | `net use` |
| Vérifier les journaux GPO | Observateur d'événements → `Applications and Services Logs > Microsoft > Windows > GroupPolicy` |
| Tester le chemin UNC | `\\serveur\partage_E` dans l'explorateur |

## Points de vigilance

| Risque | Solution |
|--------|-------------------|
| GPO non appliquée | Vérifier que l'OU cible contient bien les objets visés (users ou computers) |
| Lecteur non monté | Vérifier les droits NTFS et de partage sur `\\serveur\partage` |
| Conflit de lettre | S'assurer que E: et F: ne sont pas déjà utilisés localement (CD-ROM, etc.) |
| Action `Create` vs `Update` | `Create` ne recrée pas si existant — préférer `Update` en prod |
| Portée Computer vs User | Drive Maps = préférence **Utilisateur** — ne pas placer sous Configuration ordinateur |
| GPO liée mais non filtrée | Sans filtrage, tous les users de l'OU reçoivent les lecteurs |
