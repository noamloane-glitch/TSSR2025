# Linux - Gestion des utilisateurs

## Création et suppression

| Action | Commande |
|--------|----------|
| Créer un utilisateur | `useradd nom_user` |
| Créer avec home directory | `useradd -m nom_user` |
| Créer avec shell spécifique | `useradd -m -s /bin/bash nom_user` |
| Créer avec UID spécifique | `useradd -u 1500 nom_user` |
| Créer utilisateur système | `useradd -r nom_user` |
| Supprimer utilisateur | `userdel nom_user` |
| Supprimer avec home | `userdel -r nom_user` |

## Modification utilisateur

| Action | Commande |
|--------|----------|
| Changer le shell | `usermod -s /bin/bash nom_user` |
| Changer le home | `usermod -d /nouveau/home nom_user` |
| Renommer utilisateur | `usermod -l nouveau_nom ancien_nom` |
| Verrouiller compte | `usermod -L nom_user` |
| Déverrouiller compte | `usermod -U nom_user` |
| Changer UID | `usermod -u 1600 nom_user` |

## Mots de passe

| Action | Commande |
|--------|----------|
| Définir mot de passe | `passwd nom_user` |
| Forcer changement au login | `passwd -e nom_user` |
| Voir statut mot de passe | `passwd -S nom_user` |
| Supprimer mot de passe | `passwd -d nom_user` |
| Définir expiration (jours) | `chage -M 90 nom_user` |
| Voir politique expiration | `chage -l nom_user` |
| Définir date expiration compte | `chage -E 2026-12-31 nom_user` |
| Mot de passe sans expiration | `chage -M -1 nom_user` |

## Groupes

| Action | Commande |
|--------|----------|
| Créer un groupe | `groupadd nom_groupe` |
| Créer avec GID spécifique | `groupadd -g 2000 nom_groupe` |
| Supprimer un groupe | `groupdel nom_groupe` |
| Ajouter user à un groupe | `usermod -aG nom_groupe nom_user` |
| Ajouter à plusieurs groupes | `usermod -aG grp1,grp2 nom_user` |
| Définir groupe principal | `usermod -g nom_groupe nom_user` |
| Retirer d'un groupe | `gpasswd -d nom_user nom_groupe` |

## Sudo

| Action | Commande |
|--------|----------|
| Ajouter aux sudoers (Debian) | `usermod -aG sudo nom_user` |
| Éditer sudoers | `visudo` |
| Tester droits sudo | `sudo -l -U nom_user` |

## Consultation

| Action | Commande |
|--------|----------|
| Voir info utilisateur | `id nom_user` |
| Voir groupes d'un user | `groups nom_user` |
| Lister tous les users | `cat /etc/passwd` |
| Lister tous les groupes | `cat /etc/group` |
| Voir users connectés | `who` |
| Voir dernier login | `lastlog -u nom_user` |
| Vérifier si user existe | `getent passwd nom_user` |

## Fichiers importants

| Fichier | Contenu |
|---------|---------|
| `/etc/passwd` | Liste des utilisateurs |
| `/etc/shadow` | Mots de passe chiffrés |
| `/etc/group` | Liste des groupes |
| `/etc/gshadow` | Mots de passe groupes |
| `/etc/skel/` | Template home directory |
| `/etc/login.defs` | Politique par défaut (UID, expiration) |
