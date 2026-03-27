## 📋 Table des matières

```table-of-contents
title: 
style: nestedList # TOC style (nestedList|nestedOrderedList|inlineFirstLevel)
minLevel: 2 # Include headings from the specified level
maxLevel: 2 # Include headings up to the specified level
include: 
exclude: 
includeLinks: true # Make headings clickable
hideWhenEmpty: false # Hide TOC if no headings are found
debugInConsole: false # Print debug info in Obsidian console
```

---

## Introduction aux liens Linux

Sous Linux, il est possible de faire pointer plusieurs noms de fichiers vers un même contenu sur le disque. Ce mécanisme, appelé **lien**, est fondamental pour comprendre le fonctionnement du système de fichiers.

Il en existe deux types, aux comportements très différents :

- Le **lien physique** (hard link) : un second nom pointant directement vers les mêmes données
- Le **lien symbolique** (symlink ou soft link) : un raccourci pointant vers un chemin

> [!info] Notion d'inode Sur un système de fichiers Linux, chaque fichier est identifié par un **inode** — un numéro unique qui stocke les métadonnées (permissions, propriétaire, taille, emplacement des données). Le nom du fichier dans un répertoire n'est qu'une étiquette pointant vers cet inode.
> 
> ```bash
> # Afficher le numéro d'inode d'un fichier
> ls -li fichier.txt
> # 131074 -rw-r--r-- 1 alice alice 42 jan 10 10:00 fichier.txt
> # ^───── Numéro d'inode
> ```

---

## Lien physique (hard link)

### 🎯 Concept et utilité

Un **lien physique** crée un nouveau nom de fichier qui pointe vers le **même inode** qu'un fichier existant. Les deux noms sont strictement équivalents : il n'y a pas d'original ni de copie, juste deux entrées de répertoire qui désignent le même contenu sur le disque.

> [!example] Analogie Imaginez un livre en bibliothèque référencé sous deux titres différents dans le catalogue. Peu importe lequel vous cherchez, vous trouvez le même ouvrage physique.

**Compteur de liens (link count)** :

Le système maintient un compteur dans l'inode qui indique combien de noms pointent vers lui. Le fichier est réellement supprimé du disque uniquement quand ce compteur tombe à zéro.

```bash
# Visualisation du compteur de liens
ls -li fichier.txt lien_physique.txt
# 131074 -rw-r--r-- 2 alice alice 42 jan 10 10:00 fichier.txt
# 131074 -rw-r--r-- 2 alice alice 42 jan 10 10:00 lien_physique.txt
#                  ^ Compteur de liens = 2 (deux noms, même inode)
```

### 📝 Syntaxe et utilisation

#### Création d'un lien physique

```bash
# Syntaxe générale
ln source destination

# Exemple
ln fichier.txt lien_physique.txt

# Vérification : même inode, compteur passé à 2
ls -li fichier.txt lien_physique.txt
```

#### Comportement à la suppression

```bash
# Créer un fichier et un lien physique
echo "Contenu important" > original.txt
ln original.txt sauvegarde.txt

# Supprimer l'original
rm original.txt

# Le contenu reste accessible via l'autre nom
cat sauvegarde.txt
# Contenu important  ← toujours accessible !

# Le compteur de liens est repassé à 1
ls -li sauvegarde.txt
# 131074 -rw-r--r-- 1 alice alice 19 jan 10 10:00 sauvegarde.txt
```

> [!warning] Limitations des liens physiques
> 
> - **Impossible entre partitions** : un lien physique ne peut pas traverser deux systèmes de fichiers différents (les inodes sont locaux à leur partition)
> - **Impossible sur les répertoires** : réservé aux fichiers ordinaires (pour éviter les boucles dans l'arborescence)
> - **Risque de confusion** : modifier le contenu via l'un des noms le modifie pour tous les autres noms

#### Identifier tous les noms d'un même inode

```bash
# Trouver tous les liens physiques vers le même inode
ls -li sauvegarde.txt
# 131074 -rw-r--r-- 1 alice alice 19 jan 10 10:00 sauvegarde.txt

# Chercher tous les fichiers avec ce numéro d'inode
find / -inum 131074 2>/dev/null
```

---

## Lien symbolique (symlink)

### 🎯 Concept et utilité

Un **lien symbolique** est un fichier spécial dont le contenu est simplement un **chemin** vers une cible. Contrairement au lien physique, il ne partage pas l'inode de la cible : il possède son propre inode et stocke le chemin comme donnée.

> [!info] Le lien symbolique a son propre inode C'est pourquoi on voit un `l` en premier caractère dans `ls -l`, et pourquoi les permissions affichées sont `lrwxrwxrwx` (elles appartiennent au lien lui-même, pas à la cible).

**Avantages par rapport au lien physique** :

- Peut traverser des partitions différentes
- Peut pointer vers des répertoires
- Le chemin cible est visible et lisible
- Peut pointer vers un chemin qui n'existe pas encore

### 📝 Syntaxe et utilisation

#### Création d'un lien symbolique

```bash
# Syntaxe générale
ln -s cible nom_du_lien

# Lien vers un fichier
ln -s /etc/nginx/sites-available/monsite /etc/nginx/sites-enabled/monsite

# Lien vers un répertoire
ln -s /var/www/html /home/alice/www

# Lien relatif (cible relative au répertoire du lien)
cd /usr/local/bin
ln -s ../../opt/monapp/bin/monapp monapp
```

#### Visualisation d'un lien symbolique

```bash
# ls -l affiche la cible avec ->
ls -l /etc/nginx/sites-enabled/monsite
# lrwxrwxrwx 1 root root 34 jan 10 10:00 monsite -> /etc/nginx/sites-available/monsite
# ^── 'l' = lien symbolique

# Afficher uniquement la cible
readlink /etc/nginx/sites-enabled/monsite
# /etc/nginx/sites-available/monsite

# Afficher le chemin absolu final (résout les chaînes de liens)
readlink -f /etc/nginx/sites-enabled/monsite
```

#### Comportement à la suppression de la cible

```bash
# Créer un fichier et un lien symbolique
echo "Données" > original.txt
ln -s original.txt lien.txt

# Lire via le lien : fonctionne
cat lien.txt
# Données

# Supprimer l'original
rm original.txt

# Le lien devient "cassé" (dangling symlink)
cat lien.txt
# cat: lien.txt: Aucun fichier ou dossier de ce type

# ls -l affiche le lien cassé (cible en rouge dans la plupart des terminaux)
ls -l lien.txt
# lrwxrwxrwx 1 alice alice 11 jan 10 10:00 lien.txt -> original.txt
```

> [!warning] Lien symbolique cassé (dangling symlink) Un lien symbolique dont la cible n'existe plus est dit **cassé** ou **pendant**. Il reste visible dans le système de fichiers mais toute tentative de lecture ou d'écriture via lui échouera. Pour les détecter :
> 
> ```bash
> # Trouver tous les liens symboliques cassés dans un répertoire
> find /chemin -xtype l 2>/dev/null
> ```

#### Lien relatif vs lien absolu

```bash
# ✅ Lien absolu : robuste, fonctionne depuis n'importe où
ln -s /etc/nginx/sites-available/monsite /etc/nginx/sites-enabled/monsite

# ✅ Lien relatif : portable si on déplace la structure ensemble
cd /etc/nginx/sites-enabled/
ln -s ../sites-available/monsite monsite

# ❌ Piège : lien relatif créé depuis le mauvais répertoire
ln -s sites-available/monsite /etc/nginx/sites-enabled/monsite
# Le lien cherchera "sites-available/monsite" depuis /etc/nginx/sites-enabled/
# → chemin correct : ../sites-available/monsite
```

> [!tip] Conseil pratique Pour éviter les erreurs avec les liens relatifs, créez toujours le lien depuis le répertoire qui le contiendra, ou utilisez systématiquement des chemins absolus.

---

## Comparaison : lien physique vs lien symbolique

|Caractéristique|Lien physique|Lien symbolique|
|---|---|---|
|Partage d'inode|✅ Oui (même inode)|❌ Non (inode propre)|
|Traverse plusieurs partitions|❌ Non|✅ Oui|
|Peut pointer vers un répertoire|❌ Non|✅ Oui|
|Si la cible est supprimée|✅ Contenu conservé|❌ Lien cassé|
|Visible comme raccourci|❌ Non (transparent)|✅ Oui (lettre `l`, flèche `->`)|
|Peut pointer vers un fichier inexistant|❌ Non|✅ Oui|
|Type dans `ls -l`|`-` (fichier normal)|`l` (lien symbolique)|

> [!summary] Règle de choix rapide
> 
> - Utilisez un **lien physique** quand vous voulez une vraie redondance du nom (le contenu survit à la suppression d'un nom)
> - Utilisez un **lien symbolique** dans la quasi-totalité des autres cas : activation de configurations, raccourcis vers des répertoires, gestion de versions, etc.

---

## Cas d'usage typiques

### Le système `/etc/alternatives`

Debian et Ubuntu utilisent massivement les liens symboliques pour gérer plusieurs versions d'un même outil via le système `update-alternatives`.

```bash
# Voir le lien actuel vers la commande "editor"
ls -l /usr/bin/editor
# lrwxrwxrwx ... /usr/bin/editor -> /etc/alternatives/editor

ls -l /etc/alternatives/editor
# lrwxrwxrwx ... /etc/alternatives/editor -> /usr/bin/nano

# Lister toutes les alternatives configurées
update-alternatives --list editor

# Changer l'éditeur par défaut
sudo update-alternatives --config editor
```

### Activation de sites Nginx / Apache

```bash
# Nginx : activer un site (créer le lien dans sites-enabled)
sudo ln -s /etc/nginx/sites-available/monsite /etc/nginx/sites-enabled/monsite

# Désactiver un site (supprimer le lien sans toucher à la config)
sudo rm /etc/nginx/sites-enabled/monsite

# Apache : même logique avec a2ensite / a2dissite
sudo a2ensite monsite   # crée le lien automatiquement
sudo a2dissite monsite  # supprime le lien
```

### Gestion de versions d'applications

```bash
# Installation de deux versions d'une appli dans /opt
/opt/monapp-1.0/
/opt/monapp-2.0/

# Lien symbolique vers la version active
sudo ln -s /opt/monapp-2.0 /opt/monapp

# Les scripts appelleront /opt/monapp/bin/monapp sans se soucier de la version
ls -l /opt/monapp
# lrwxrwxrwx ... /opt/monapp -> /opt/monapp-2.0

# Passer à une autre version : un seul changement suffit
sudo ln -snf /opt/monapp-1.0 /opt/monapp
```

> [!tip] L'option `-n` avec `ln -s` L'option `-n` (ou `--no-dereference`) est indispensable quand la destination est déjà un lien symbolique vers un répertoire. Sans elle, le nouveau lien serait créé _à l'intérieur_ du répertoire cible au lieu de le remplacer.
> 
> ```bash
> # ✅ Remplace le lien existant
> ln -snf /opt/monapp-1.0 /opt/monapp
> 
> # ❌ Sans -n : crée un lien DANS /opt/monapp/ si c'est déjà un lien vers un répertoire
> ln -sf /opt/monapp-1.0 /opt/monapp
> ```

### Répertoires dans /bin → /usr/bin

Sur les distributions modernes (Debian 10+, Ubuntu 20.04+), `/bin`, `/sbin` et `/lib` sont devenus des liens symboliques vers `/usr/bin`, `/usr/sbin` et `/usr/lib` pour unifier la hiérarchie.

```bash
# Vérification
ls -ld /bin /sbin /lib
# lrwxrwxrwx 1 root root 7 ... /bin -> usr/bin
# lrwxrwxrwx 1 root root 8 ... /sbin -> usr/sbin
# lrwxrwxrwx 1 root root 7 ... /lib -> usr/lib
```

---

## Gestion et manipulation avancées

### Supprimer un lien sans toucher la cible

```bash
# ✅ Supprime le lien uniquement
rm lien_symbolique

# ⚠️ Attention avec les répertoires : ne pas mettre de / à la fin !
rm lien_vers_repertoire       # ✅ Supprime le lien
rm lien_vers_repertoire/      # ❌ Opère sur le contenu du répertoire cible !
```

### Copier vs suivre un lien

```bash
# cp suit le lien par défaut : copie le CONTENU de la cible
cp lien.txt copie.txt          # copie le fichier pointé

# cp -P préserve le lien lui-même
cp -P lien.txt copie_du_lien.txt   # copie le lien symbolique

# rsync : -l préserve les liens symboliques (comportement par défaut avec -a)
rsync -a source/ destination/
```

### Trouver les liens symboliques

```bash
# Tous les liens symboliques d'un répertoire
find /etc -type l

# Liens symboliques cassés uniquement
find /etc -xtype l 2>/dev/null

# Afficher la cible pour chaque lien trouvé
find /etc -type l -exec ls -l {} \;
```

---

## Pièges courants

### ⚠️ Confondre la suppression du lien et de la cible

```bash
# ⚠️ Supprimer la cible ne supprime pas le lien
rm /etc/nginx/sites-available/monsite
# Le lien /etc/nginx/sites-enabled/monsite est maintenant cassé !

# ✅ Toujours supprimer le lien ET la cible si besoin
sudo rm /etc/nginx/sites-enabled/monsite          # lien d'abord
sudo rm /etc/nginx/sites-available/monsite        # puis la config
```

### ⚠️ Lien relatif créé depuis le mauvais répertoire

```bash
# Contexte : on veut créer /usr/local/bin/monapp → /opt/monapp/bin/monapp
# ❌ Depuis /home/alice : le chemin relatif sera résolu depuis /usr/local/bin/
ln -s ../../opt/monapp/bin/monapp /usr/local/bin/monapp   # Correct !

# Vérification systématique après création
ls -l /usr/local/bin/monapp
readlink -f /usr/local/bin/monapp   # Doit retourner le chemin absolu attendu
```

### ⚠️ chmod sur un lien symbolique

```bash
# chmod sur un lien symbolique agit sur la CIBLE, pas sur le lien
chmod 755 lien.txt    # Modifie les permissions de la cible

# Les permissions du lien lui-même (lrwxrwxrwx) ne changent jamais
# et sont sans effet sur le contrôle d'accès
```

### ⚠️ Lien physique sur un répertoire

```bash
# ❌ Interdit : les liens physiques sur répertoires sont bloqués par le noyau
ln /home/alice /home/alice_lien
# ln: '/home/alice': liaison physique non permise pour un répertoire

# ✅ Utiliser un lien symbolique à la place
ln -s /home/alice /home/alice_lien
```

> [!warning] Vérifier un lien avant de l'utiliser
> 
> ```bash
> # Vérifier qu'un lien existe et que sa cible est accessible
> if [ -L lien.txt ] && [ -e lien.txt ]; then
>     echo "Lien valide"
> elif [ -L lien.txt ]; then
>     echo "Lien cassé (cible manquante)"
> else
>     echo "Pas un lien symbolique"
> fi
> ```

---

## Bonnes pratiques

### 🎯 Documenter la raison d'un lien

```bash
# Dans un script, toujours expliquer pourquoi on crée un lien
# Activer le site "monsite" pour Nginx (lien vers sites-enabled)
sudo ln -s /etc/nginx/sites-available/monsite /etc/nginx/sites-enabled/monsite
```

### 🎯 Vérifier après création

```bash
# Après chaque création de lien, vérifier que la cible est accessible
ln -s /opt/monapp-2.0 /opt/monapp
ls -l /opt/monapp          # Voir le lien
readlink -f /opt/monapp    # Résoudre le chemin absolu
test -e /opt/monapp && echo "OK" || echo "Lien cassé"
```

### 🎯 Préférer les chemins absolus dans les scripts

```bash
# ✅ Robuste : fonctionne quel que soit le répertoire courant
ln -s /chemin/absolu/vers/cible /chemin/absolu/vers/lien

# ⚠️ Fragile : dépend du répertoire depuis lequel le script est exécuté
ln -s cible lien
```

### 🎯 Utiliser les liens pour la gestion de configuration

```bash
# Bonne pratique : conserver les configs dans un répertoire versionné
# et utiliser des liens symboliques pour les activer
mkdir -p ~/dotfiles
mv ~/.bashrc ~/dotfiles/bashrc
ln -s ~/dotfiles/bashrc ~/.bashrc

# Ainsi les configs peuvent être versionnées avec Git
# et les liens activent ou désactivent les configurations
```

### 🎯 Nettoyer les liens cassés

```bash
# Identifier les liens cassés
find /etc -xtype l 2>/dev/null

# Supprimer tous les liens cassés dans un répertoire (avec confirmation)
find /etc/nginx/sites-enabled -xtype l -exec ls -l {} \;
# Vérifier la liste, puis supprimer
find /etc/nginx/sites-enabled -xtype l -delete
```

---

## 🎯 Points clés à retenir

> [!summary] Résumé des liens Linux
> 
> **Lien physique** :
> 
> - Même inode que l'original → même fichier, deux noms
> - Le contenu survit à la suppression de l'un des noms
> - Limité aux fichiers, impossible entre partitions
> - Commande : `ln source destination`
> 
> **Lien symbolique** :
> 
> - Inode propre contenant un chemin vers la cible
> - Devient cassé si la cible est supprimée
> - Peut pointer vers fichiers, répertoires, autres partitions
> - Commande : `ln -s cible nom_du_lien`
> 
> **Commandes essentielles** :
> 
> ```bash
> ln fichier lien_physique          # Lien physique
> ln -s cible lien_symbolique       # Lien symbolique
> ls -li fichier                    # Voir l'inode et le compteur de liens
> readlink lien                     # Lire la cible d'un lien symbolique
> readlink -f lien                  # Résoudre le chemin absolu complet
> find /chemin -type l              # Trouver les liens symboliques
> find /chemin -xtype l             # Trouver les liens symboliques cassés
> ```

> [!tip] Règle d'or Dans 95 % des cas, vous utiliserez des **liens symboliques**. Réservez les liens physiques aux rares situations où vous avez besoin qu'un contenu survive à la suppression de son nom d'origine (sauvegardes, journaux de transactions).

---