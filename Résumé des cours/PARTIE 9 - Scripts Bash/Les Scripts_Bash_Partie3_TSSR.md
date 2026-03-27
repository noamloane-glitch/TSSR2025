# Les scripts Bash - Partie 3
## Document de révision TSSR - Titre RNCP

---

**Formation** : Technicien Supérieur Systèmes et Réseaux (TSSR)  
**Sujet** : Structuration du code - Fonctions et bonnes pratiques  
**Date** : Novembre 2025  
**Type** : Synthèse de cours complète

---

## 📋 Sommaire

1. [[#Introduction|Introduction]]
2. [[#Les fonctions|Les fonctions]]
   - [[#Définition fonction|Définition]]
   - [[#Déclarer fonction|Déclaration]]
   - [[#Paramètres fonction|Paramètres]]
   - [[#Valeur de retour|Valeur de retour]]
   - [[#Périmètre variables|Périmètre (portée)]]
3. [[#Usages et bonnes pratiques|Usages et bonnes pratiques]]
   - [[#Shebang|Shebang]]
   - [[#Commentaires|Commentaires]]
   - [[#Sudo dans scripts|Sudo dans les scripts]]
   - [[#Utilisation variables|Utilisation des variables]]
   - [[#Gestion erreurs|Gestion des erreurs]]
   - [[#Utilisation fonctions|Utilisation des fonctions]]
   - [[#Fichiers log|Fichiers de log]]
   - [[#Outils vérification|Outils de vérification]]
4. [[#Ressources|Ressources et références]]
5. [[#Points clés à retenir|Points clés]]
6. [[#Glossaire technique|Glossaire]]

---

## Introduction

> [!abstract] Vue d'ensemble
> Cette **partie 3** du cours sur les scripts Bash se concentre sur la **structuration du code** et les **bonnes pratiques de développement**. Les fonctions sont au cœur de cette approche, permettant de créer des scripts **maintenables**, **réutilisables** et **professionnels**.

### Pourquoi structurer son code ?

En tant que **TSSR**, vous devez :

- Créer des scripts **maintenables** et **évolutifs**
- **Réutiliser** du code efficacement
- Faciliter le **débogage** et la correction d'erreurs
- Travailler en **équipe** avec des scripts compréhensibles
- Respecter les **standards professionnels** du métier

**Objectifs de cette partie** :
- ✅ Maîtriser les fonctions Bash
- ✅ Appliquer les bonnes pratiques de développement
- ✅ Écrire du code propre et sécurisé
- ✅ Documenter correctement ses scripts

---

## Les fonctions

### Définition fonction

> [!quote] Définition
> Une **fonction** est un **bloc de code nommé** qu'on déclare pour pouvoir l'utiliser plus tard, éventuellement **plusieurs fois**.

> [!important] Avantages des fonctions

**Structuration du code** :
- ✅ Organisation logique en blocs réutilisables
- ✅ Code plus lisible et compréhensible
- ✅ Séparation des responsabilités

**Réutilisation** :
- ✅ Éviter la duplication de code
- ✅ Appeler une fonction autant de fois que nécessaire
- ✅ Modifier une fois, impact partout

**Maintenance** :
- ✅ Corrections centralisées
- ✅ Tests plus faciles
- ✅ Évolution simplifiée

> [!warning] Ordre de déclaration
> **IMPORTANT** : Les fonctions doivent être **déclarées AVANT d'être appelées**.
> 
> En général, on place les fonctions en **début de script** (après le shebang et les commentaires d'en-tête).

---

### Déclarer fonction

> [!info] Syntaxes de déclaration

Il existe **deux syntaxes** pour déclarer une fonction en Bash :

#### Syntaxe 1 (avec mot-clé `function`)

```bash
function nom()
{
    instructions
}
```

#### Syntaxe 2 (sans mot-clé `function`)

```bash
nom()
{
    instructions
}
```

> [!tip] Quelle syntaxe choisir ?
> - **Syntaxe 1** : Plus explicite, recommandée pour la lisibilité
> - **Syntaxe 2** : Plus portable (POSIX), préférée par certains
> 
> **Les deux sont valides**, choisissez celle que vous préférez et restez cohérent dans vos scripts.

---

#### Exemple complet

> [!example] Ma première fonction

```bash
#!/bin/bash

# Declaring the hello function
function hello()
{
    echo "Hi folks !"
}

# Appel de la fonction
hello
echo "and again"
hello
```

**Exécution** :
```bash
wilder@host:~$ ./script.sh
Hi folks !
and again
Hi folks !
```

**Analyse** :
1. La fonction `hello` est **déclarée** en début de script
2. Premier appel : affiche "Hi folks !"
3. Affichage de "and again"
4. Deuxième appel : affiche à nouveau "Hi folks !"

> [!success] À retenir
> - Une fonction peut être appelée **plusieurs fois**
> - L'appel se fait simplement en écrivant le **nom** de la fonction
> - Les accolades `{}` délimitent le **corps** de la fonction

---

### Paramètres fonction

> [!important] Des fonctions adaptables

Les fonctions Bash peuvent accepter des **arguments** (paramètres) pour être plus flexibles.

**Principe** :
- Un appel de fonction peut être **suivi d'arguments**
- On les récupère dans la fonction comme les **paramètres d'un script** : `$1`, `$2`, `$3`, etc.
- `$#` = nombre d'arguments
- `$@` = tous les arguments

---

#### Exemple avec paramètres

> [!example] Fonction greet avec paramètre optionnel

```bash
#!/bin/bash

function hello()
{
    echo "Hi folks !"
}

greet()
{
    if [ $# -gt 0 ]
    then
        echo "Hi $1"
    else
        hello
    fi
}

# Appels
greet wilder    # Sortie : Hi wilder
greet           # Sortie : Hi folks !
```

**Analyse** :
- `greet wilder` : Passe "wilder" comme `$1`, affiche "Hi wilder"
- `greet` : Aucun argument, appelle `hello()` qui affiche "Hi folks !"

---

#### Paramètres multiples

> [!example] Fonction avec plusieurs paramètres

```bash
#!/bin/bash

afficher_info()
{
    local nom="$1"
    local age="$2"
    local ville="$3"
    
    echo "Nom : $nom"
    echo "Âge : $age ans"
    echo "Ville : $ville"
}

afficher_info "Alice" 25 "Paris"
afficher_info "Bob" 30 "Lyon"
```

**Sortie** :
```
Nom : Alice
Âge : 25 ans
Ville : Paris
Nom : Bob
Âge : 30 ans
Ville : Lyon
```

> [!tip] Bonnes pratiques avec paramètres
> - Toujours mettre les variables entre **guillemets** : `"$1"`, `"$2"`
> - Vérifier le **nombre d'arguments** avec `$#`
> - Utiliser des **noms de variables locales** explicites
> - Documenter les paramètres attendus dans un commentaire

---

### Valeur de retour

> [!important] Retourner des valeurs

Il existe **deux façons** de sortir une valeur depuis une fonction :

#### 1. `return <code>` : Code de sortie numérique

**Caractéristiques** :
- Renvoie un **code de sortie** entre **0 et 255**
- **0** = succès
- **1-255** = erreur (différents codes d'erreur)
- Récupéré via `$?`

**Exemple** :
```bash
#!/bin/bash

verifier_fichier()
{
    if [ -f "$1" ]
    then
        return 0  # Fichier existe
    else
        return 1  # Fichier n'existe pas
    fi
}

verifier_fichier "/etc/passwd"
if [ $? -eq 0 ]
then
    echo "Le fichier existe"
else
    echo "Le fichier n'existe pas"
fi
```

---

#### 2. `echo <valeur>` : Valeur texte

**Caractéristiques** :
- Renvoie une **chaîne de caractères** (ou nombre sous forme texte)
- Récupéré via **substitution de commande** : `$(fonction)`
- Permet de retourner des **données complexes**

**Exemple** :
```bash
#!/bin/bash

fonc_calcul()
{
    somme=$(($1 + $2))
    echo $somme
}

echo "Donne 2 nombres"
read -p "Le premier ? " nbr1
read -p "Le second ? " nbr2

resultat=$(fonc_calcul $nbr1 $nbr2)

echo "Le résultat de la somme de $nbr1 et $nbr2 est $resultat"
exit 0
```

**Exécution** :
```bash
wilder@host:~$ ./script.sh
Donne 2 nombres
Le premier ? 15
Le second ? 27
Le résultat de la somme de 15 et 27 est 42
```

---

#### Comparaison des deux méthodes

> [!example] Quand utiliser quelle méthode ?

| Méthode | Usage | Avantages | Inconvénients |
|---------|-------|-----------|---------------|
| **`return`** | Code de succès/erreur | Standard Unix, simple | Limité à 0-255, numérique uniquement |
| **`echo`** | Retour de données | Flexible, texte/nombres | Peut être pollué par d'autres echo |

**Recommandations** :
- **`return`** : Pour les codes d'erreur et le statut de réussite
- **`echo`** : Pour retourner des valeurs calculées, du texte, des données

> [!warning] Piège avec echo
> Si votre fonction contient plusieurs `echo`, **tous seront capturés** par `$()`.
> 
> Solution : Rediriger les messages de debug vers stderr : `echo "Debug" >&2`

---

### Périmètre variables

> [!important] Portée des variables (scope)

**Définition** :
Le **périmètre** (ou **portée** / **scope**) d'une variable définit où elle est **accessible** dans le script.

#### Variables globales (par défaut)

> [!info] Comportement par défaut

Par défaut, dans un script Bash, une variable est **globale** :
- Sa valeur est connue dans **l'ensemble du script**
- Accessible partout (dans et hors des fonctions)
- Modifications visibles partout

---

#### Variables locales (mot-clé `local`)

> [!success] Bonne pratique : variables locales

Pour qu'une variable reste **confinée** dans la fonction où elle est déclarée, utiliser le mot-clé **`local`**.

**Avantages** :
- ✅ Évite les **conflits de noms**
- ✅ Isole les données de la fonction
- ✅ Meilleure **maintenabilité**

---

#### Exemple comparatif

> [!example] Impact de `local`

**Sans `local` (variable globale)** :

```bash
#!/bin/bash

test_fonc()
{
    var="Bonjour"    # Modifie la variable globale
}

var="Au revoir"
test_fonc
echo "$var"          # Affiche : Bonjour
exit 0
```

**Sortie** : `Bonjour` (la fonction a modifié la variable globale)

---

**Avec `local` (variable locale)** :

```bash
#!/bin/bash

test_fonc()
{
    local var="Bonjour"    # Variable locale à la fonction
}

var="Au revoir"
test_fonc
echo "$var"                 # Affiche : Au revoir
exit 0
```

**Sortie** : `Au revoir` (la variable locale n'affecte pas la globale)

---

#### Visualisation de la portée

```
┌────────────────────────────────────┐
│        SCRIPT (portée globale)     │
│                                    │
│  var="Au revoir"  ← Global         │
│                                    │
│  ┌──────────────────────────────┐ │
│  │  FONCTION (portée locale)    │ │
│  │                              │ │
│  │  local var="Bonjour" ← Local│ │
│  │                              │ │
│  └──────────────────────────────┘ │
│                                    │
│  echo "$var" → "Au revoir"         │
└────────────────────────────────────┘
```

> [!tip] Bonne pratique
> **Toujours** utiliser `local` pour les variables à l'intérieur des fonctions, sauf si vous avez besoin de modifier une variable globale.

---

## Usages et bonnes pratiques

> [!abstract] Vue d'ensemble
> Les bonnes pratiques permettent de créer des scripts **professionnels**, **maintenables** et **sécurisés**. Elles sont essentielles dans le métier de TSSR.

---

### Shebang

> [!important] 1. Mettre un shebang

**Définition** :
Le **shebang** (ou **hashbang**) est la première ligne d'un script qui spécifie l'**interpréteur** à utiliser.

#### Syntaxe standard

```bash
#!/bin/bash
echo "Hello !"
```

**Explication** :
- `#!` : Caractères magiques indiquant un shebang
- `/bin/bash` : Chemin absolu vers l'interpréteur Bash

---

#### Syntaxe portable (recommandée)

```bash
#!/usr/bin/env bash
echo "Hello !"
```

**Avantages** :
- ✅ **Meilleure portabilité** entre systèmes
- ✅ Utilise la version de Bash dans le `PATH`
- ✅ Fonctionne même si Bash n'est pas dans `/bin/`

> [!tip] Recommandation
> Préférer `#!/usr/bin/env bash` pour une meilleure portabilité, sauf contraintes spécifiques.

---

#### Autres shebangs courants

| Shebang | Usage |
|---------|-------|
| `#!/bin/bash` | Script Bash (chemin fixe) |
| `#!/usr/bin/env bash` | Script Bash (portable) |
| `#!/bin/sh` | Script POSIX (compatible) |
| `#!/usr/bin/env python3` | Script Python 3 |
| `#!/usr/bin/env node` | Script Node.js |

> [!warning] Attention
> Sans shebang, le script sera exécuté avec le **shell par défaut** de l'utilisateur (qui peut ne pas être Bash).

---

### Commentaires

> [!important] 2. Mettre des commentaires

**Principe** :
Les commentaires expliquent **ce que fait le code** et **pourquoi**.

#### Commentaires simples

```bash
#!/bin/bash

# Mise à jour de la liste des paquets
apt update

# Mise à jour du système
apt upgrade -y
```

> [!quote] Citation
> "Le code explique comment, les commentaires expliquent pourquoi."

---

#### En-tête de script

> [!example] Commentaire d'en-tête professionnel

```bash
#!/bin/bash

######################################
# addUser.sh
# Utilité : Ce script sert à créer des utilisateurs passés en argument
# Usage : addUser.sh utilisateur1 utilisateur2 …
# Auteur : John DOE <j.doe@wcs.com>
# Mise à jour le : 01/02/2025
######################################
```

**Informations à inclure** :
- ✅ **Nom** du script
- ✅ **Utilité** / description courte
- ✅ **Usage** / syntaxe d'exécution
- ✅ **Auteur** et contact
- ✅ **Date** de création/mise à jour
- ✅ **Version** (optionnel)
- ✅ **Dépendances** (optionnel)

---

#### Bonnes pratiques de commentaires

> [!tip] Quand commenter ?

**À commenter** :
- ✅ **Logique complexe** difficile à comprendre
- ✅ **Choix techniques** non évidents
- ✅ **Workarounds** (contournements)
- ✅ **TODO** et **FIXME**
- ✅ **Sections** du script

**À ne pas commenter** :
- ❌ Code **évident** (ex: `x = x + 1  # Incrémente x`)
- ❌ **Répétition** du code en français
- ❌ Commentaires **obsolètes** ou **faux**

> [!warning] Maintenir les commentaires
> Un commentaire **faux** est **pire** que pas de commentaire. Mettez à jour les commentaires quand vous modifiez le code !

---

### Sudo dans scripts

> [!warning] 3. Ne pas mettre sudo dans les scripts

**Principe** :
Il vaut mieux **exécuter un script avec sudo** que mettre `sudo` **dans** le script.

#### Mauvaise pratique ❌

```bash
#!/bin/bash
# Mise à jour de la liste des paquets et du système
sudo apt update && sudo apt upgrade -y
```

**Problèmes** :
- ❌ Perte de contrôle sur les permissions
- ❌ Élévation de privilèges sur tout le script
- ❌ Difficile de tracer qui a fait quoi
- ❌ Risque de sécurité

---

#### Bonne pratique ✅

**Script sans sudo** :
```bash
#!/bin/bash
# Mise à jour de la liste des paquets et du système
apt update && apt upgrade -y
```

**Exécution** :
```bash
sudo ./script.sh
```

**Avantages** :
- ✅ **Plus de contrôle** : l'utilisateur sait qu'il exécute avec sudo
- ✅ **Meilleure sécurité** : pas d'élévation sur parties non-essentielles
- ✅ Possibilité d'exécuter avec un **utilisateur précis** : `sudo -u <user> ./script.sh`

---

#### Vérification d'exécution en sudo

> [!example] Bonus : Forcer l'exécution avec sudo

```bash
#!/bin/bash

# Vérification d'exécution en sudo
if [[ $EUID -ne 0 ]]
then
    echo "Exécution du script obligatoire en sudo" >&2
    exit 1
fi

# Mise à jour de la liste des paquets
apt update
exit 0
```

**Explication** :
- `$EUID` : Effective User ID (ID de l'utilisateur effectif)
- `0` : ID de l'utilisateur root
- Si `$EUID -ne 0` : pas root → message d'erreur + sortie

**Exécution** :
```bash
wilder@host:~$ ./script.sh
Exécution du script obligatoire en sudo

wilder@host:~$ sudo ./script.sh
[Exécution normale du script]
```

---

### Utilisation variables

> [!important] 4. Utiliser des variables

**Principe** :
Les variables rendent le code **flexible**, **modifiable** et **maintenable**.

#### Exemple basique

```bash
#!/bin/bash

nom="$1"
age="$2"

read -p "Métier : " metier

echo "$nom a $age ans et son métier est $metier"
```

**Exécution** :
```bash
./script.sh wilder 25
Métier : Développeur
wilder a 25 ans et son métier est Développeur
```

---

#### Guillemets autour des variables

> [!warning] Toujours mettre des guillemets !

**Problème sans guillemets** :
```bash
var="string with spaces"
[ $var = "string with spaces" ] && echo "OK" || echo "KO"
```

**Erreur** :
```bash
bash: [: too many arguments
KO
```

**Raison** : La variable `$var` est expansée en plusieurs mots → trop d'arguments pour `[`.

---

**Solution avec guillemets** ✅ :

```bash
var="string with spaces"
[ "$var" = "string with spaces" ] && echo "OK" || echo "KO"
```

**Sortie** : `OK`

**Ou avec `[[ ]]`** (recommandé) :
```bash
var="string with spaces"
[[ $var = "string with spaces" ]] && echo "OK" || echo "KO"
```

**Sortie** : `OK`

> [!tip] Règle d'or
> **Toujours** mettre les variables entre **guillemets doubles** : `"$var"`
> 
> Exception : `[[ ]]` gère mieux les espaces que `[ ]`

---

#### Variables avec valeurs par défaut

> [!example] Valeurs par défaut

```bash
#!/bin/bash

# Si $1 n'est pas défini, utiliser "Invité"
nom="${1:-Invité}"

# Si $2 n'est pas défini, utiliser "Inconnu"
metier="${2:-Inconnu}"

echo "Bonjour $nom, métier : $metier"
```

**Exécution** :
```bash
./script.sh Alice Développeuse
# Sortie : Bonjour Alice, métier : Développeuse

./script.sh Bob
# Sortie : Bonjour Bob, métier : Inconnu

./script.sh
# Sortie : Bonjour Invité, métier : Inconnu
```

---

### Gestion erreurs

> [!important] 5. Gérer les erreurs

**Principe** :
Erreurs de saisie, mauvais typage, fichiers inexistants… doivent être **gérées** pour éviter des sorties non-prévues.

#### Avec l'opérateur `||`

```bash
cd dossier10 2>/dev/null && echo "Déplacement dans le dossier" || echo "Dossier inexistant"
```

**Explication** :
- `cd dossier10` : Tentative de changement de répertoire
- `2>/dev/null` : Redirection des erreurs vers `/dev/null` (silence)
- `&&` : Si succès, afficher "Déplacement dans le dossier"
- `||` : Sinon, afficher "Dossier inexistant"

---

#### Vérifications préalables

> [!example] Vérifier avant d'agir

```bash
#!/bin/bash

fichier="$1"

# Vérifier que le fichier existe
if [[ ! -f "$fichier" ]]
then
    echo "Erreur : Le fichier '$fichier' n'existe pas" >&2
    exit 1
fi

# Traiter le fichier
echo "Traitement de $fichier..."
```

---

#### Codes de sortie

> [!info] Utiliser les codes de sortie

**Convention Unix** :
- **0** : Succès
- **1-255** : Erreur (différents codes)

**Codes courants** :

| Code | Signification |
|------|---------------|
| **0** | Succès |
| **1** | Erreur générale |
| **2** | Mauvaise utilisation (arguments) |
| **126** | Commande non exécutable |
| **127** | Commande introuvable |
| **130** | Interrompu par Ctrl+C |

**Exemple** :
```bash
#!/bin/bash

if [ $# -eq 0 ]
then
    echo "Usage : $0 <fichier>" >&2
    exit 2    # Code 2 = mauvaise utilisation
fi

# Reste du script...
exit 0    # Code 0 = succès
```

---

#### Avec `case` pour gérer plusieurs cas

```bash
#!/bin/bash

case "$1" in
    start)
        echo "Démarrage du service..."
        ;;
    stop)
        echo "Arrêt du service..."
        ;;
    restart)
        echo "Redémarrage du service..."
        ;;
    *)
        echo "Erreur : Commande non reconnue" >&2
        echo "Usage : $0 {start|stop|restart}" >&2
        exit 1
        ;;
esac
```

---

### Utilisation fonctions

> [!important] 6. Utiliser des fonctions

**Avantages** :
- ✅ Code plus **lisible**
- ✅ **Réutilisation** (évite duplication)
- ✅ **Maintenance** facilitée
- ✅ **Tests** plus faciles

#### Exemple sans fonctions (répétitif)

```bash
#!/bin/bash

# Calcul 1
nombre1=5
nombre2=10
resultat=$((nombre1 + nombre2))
echo "Résultat 1 : $resultat"

# Calcul 2
nombre1=1
nombre2=3
resultat=$((nombre1 + nombre2))
echo "Résultat 2 : $resultat"

# Calcul 3
nombre1=10
nombre2=200
resultat=$((nombre1 + nombre2))
echo "Résultat 3 : $resultat"
```

---

#### Exemple avec fonction (optimisé)

```bash
#!/bin/bash

# Fonction de calcul
calculer()
{
    local nombre1="$1"
    local nombre2="$2"
    local resultat=$((nombre1 + nombre2))
    echo "$resultat"
}

# Appels
echo "Résultat 1 : $(calculer 5 10)"
echo "Résultat 2 : $(calculer 1 3)"
echo "Résultat 3 : $(calculer 10 200)"
```

**Avantages** :
- ✅ **Moins de lignes** (pas de duplication)
- ✅ Modification **centralisée** (un seul endroit)
- ✅ **Lisibilité** améliorée

---

### Fichiers log

> [!important] 7. Utiliser des fichiers de log

**Principe** :
Journaliser l'activité du script pour faciliter le **débogage** et l'**audit**.

#### Exemple simple

```bash
#!/bin/bash

LOG_FILE="/var/log/mon_script.log"

# Fonction de log
log()
{
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" >> "$LOG_FILE"
}

# Utilisation
log "Démarrage du script"

apt update
if [ $? -eq 0 ]
then
    log "Mise à jour réussie"
else
    log "ERREUR : Échec de la mise à jour"
fi

log "Fin du script"
```

**Contenu de `/var/log/mon_script.log`** :
```
[2025-11-21 14:30:15] Démarrage du script
[2025-11-21 14:30:22] Mise à jour réussie
[2025-11-21 14:30:22] Fin du script
```

---

#### Redirection vers log ET console

> [!example] Afficher et logger

```bash
#!/bin/bash

LOG_FILE="/var/log/mon_script.log"

# Fonction log vers fichier ET console
log()
{
    local message="[$(date '+%Y-%m-%d %H:%M:%S')] $*"
    echo "$message" | tee -a "$LOG_FILE"
}

log "Démarrage du script"
log "Traitement en cours..."
log "Fin du script"
```

**`tee -a`** :
- Affiche sur **stdout** (console)
- ET ajoute (`-a`) au **fichier**

---

#### Niveaux de log

> [!tip] Distinguer les niveaux de gravité

```bash
#!/bin/bash

LOG_FILE="/var/log/mon_script.log"

log_info()  { echo "[INFO]  [$(date '+%Y-%m-%d %H:%M:%S')] $*" >> "$LOG_FILE"; }
log_warn()  { echo "[WARN]  [$(date '+%Y-%m-%d %H:%M:%S')] $*" >> "$LOG_FILE"; }
log_error() { echo "[ERROR] [$(date '+%Y-%m-%d %H:%M:%S')] $*" >> "$LOG_FILE"; }

# Utilisation
log_info "Démarrage du script"
log_warn "Attention : Le disque est presque plein"
log_error "Erreur critique : Impossible de se connecter à la base"
```

**Sortie** :
```
[INFO]  [2025-11-21 14:30:15] Démarrage du script
[WARN]  [2025-11-21 14:30:16] Attention : Le disque est presque plein
[ERROR] [2025-11-21 14:30:17] Erreur critique : Impossible de se connecter à la base
```

---

### Outils vérification

> [!important] 8. Se faire aider par des outils

Il existe des **outils** pour vérifier et formater automatiquement vos scripts Bash.

#### ShellCheck

> [!success] Linter pour Bash (analyse statique)

**Installation** :
```bash
# Debian/Ubuntu
sudo apt install shellcheck

# macOS
brew install shellcheck
```

**Utilisation** :
```bash
shellcheck mon_script.sh
```

**Exemple de sortie** :
```
In mon_script.sh line 5:
if [ $var = "test" ]
     ^-- SC2086: Double quote to prevent globbing and word splitting.

In mon_script.sh line 10:
cd $HOME/dossier
   ^-- SC2086: Double quote to prevent globbing and word splitting.
```

**Avantages** :
- ✅ Détecte les **erreurs courantes**
- ✅ Suggère des **améliorations**
- ✅ Explique les **bonnes pratiques**
- ✅ Intégration dans les **éditeurs** (VS Code, Vim, Sublime)

---

#### shfmt

> [!success] Formateur pour Bash

**Installation** :
```bash
# Avec Go
go install mvdan.cc/sh/v3/cmd/shfmt@latest

# Ou téléchargement binaire
wget https://github.com/mvdan/sh/releases/download/v3.7.0/shfmt_v3.7.0_linux_amd64
chmod +x shfmt_v3.7.0_linux_amd64
sudo mv shfmt_v3.7.0_linux_amd64 /usr/local/bin/shfmt
```

**Utilisation** :
```bash
# Afficher le code formaté
shfmt mon_script.sh

# Formater en place (modifier le fichier)
shfmt -w mon_script.sh

# Formater avec indentation de 4 espaces
shfmt -i 4 -w mon_script.sh
```

**Avantages** :
- ✅ **Indentation** automatique
- ✅ **Cohérence** du style
- ✅ Gain de **temps**

---

#### Intégration dans VS Code

> [!tip] Configuration recommandée

**Extensions VS Code** :
- **shellcheck** : Analyse en temps réel
- **shell-format** : Formatage automatique
- **Bash IDE** : Auto-complétion, snippets

**Configuration** (`.vscode/settings.json`) :
```json
{
  "shellcheck.enable": true,
  "shellformat.flag": "-i 4",
  "[shellscript]": {
    "editor.formatOnSave": true
  }
}
```

---

## Ressources

> [!note] Documentation et références

### Documentation officielle

**Documentation Bash** :
- **GNU Bash Manual** : https://www.gnu.org/software/bash/manual/
- **Bash Reference Manual** : https://www.gnu.org/software/bash/manual/bash.html

### Tutoriels et guides

**Français** :
- **Wikibooks - Programmation Bash** : https://fr.wikibooks.org/wiki/Programmation_Bash
- **Le Guide du Shell Bash** : Ressource francophone complète

**Anglais** :
- **Bash Hackers Wiki** : https://wiki.bash-hackers.org/
- **The Bash Guide (Greg)** : https://mywiki.wooledge.org/BashGuide
- **Advanced Bash-Scripting Guide** : https://tldp.org/LDP/abs/html/

### Outils en ligne

**ExplainShell** :
- URL : https://explainshell.com/
- Explique ligne par ligne les commandes Bash
- Décompose les options et arguments

**ShellCheck Online** :
- URL : https://www.shellcheck.net/
- Vérification en ligne sans installation

---

## Points clés à retenir

> [!success] Synthèse pour le titre RNCP

### Fonctions Bash

**Définition** :
- Bloc de code nommé **réutilisable**
- Déclaration **avant** l'appel
- Deux syntaxes possibles

**Syntaxe** :
```bash
function nom()
{
    instructions
}

# OU

nom()
{
    instructions
}
```

**Paramètres** :
- `$1`, `$2`, `$3`, ... = arguments
- `$#` = nombre d'arguments
- `$@` = tous les arguments

**Valeurs de retour** :
- `return <code>` : Code de sortie 0-255
- `echo <valeur>` : Retour de texte/données

**Portée (scope)** :
- Variables **globales** par défaut
- `local` pour variables **locales** (recommandé)

---

### Bonnes pratiques essentielles

#### 1. Shebang

```bash
#!/usr/bin/env bash    # Portable (recommandé)
#!/bin/bash            # Standard
```

---

#### 2. Commentaires

**En-tête de script** :
```bash
######################################
# nom_script.sh
# Utilité : Description du script
# Usage : ./script.sh [options]
# Auteur : Nom <email>
# Date : JJ/MM/AAAA
######################################
```

**Commentaires inline** :
- Expliquer le **pourquoi**, pas le **comment**
- Commenter la **logique complexe**

---

#### 3. Sudo

**À éviter** ❌ :
```bash
#!/bin/bash
sudo apt update
```

**Recommandé** ✅ :
```bash
#!/bin/bash
apt update
```

**Exécution** : `sudo ./script.sh`

**Vérification** (optionnel) :
```bash
if [[ $EUID -ne 0 ]]; then
    echo "Exécution en sudo obligatoire" >&2
    exit 1
fi
```

---

#### 4. Variables

**Toujours entre guillemets** :
```bash
nom="$1"
echo "$nom"
[ "$var" = "valeur" ]
```

**Valeurs par défaut** :
```bash
nom="${1:-Invité}"
```

---

#### 5. Gestion des erreurs

**Codes de sortie** :
- `0` = succès
- `1-255` = erreur

**Opérateurs** :
```bash
commande && echo "Succès" || echo "Échec"
```

**Vérifications** :
```bash
if [[ ! -f "$fichier" ]]; then
    echo "Erreur : fichier inexistant" >&2
    exit 1
fi
```

---

#### 6. Fonctions

**Avantages** :
- Réutilisation du code
- Maintenance facilitée
- Lisibilité améliorée

**Exemple** :
```bash
calculer()
{
    local result=$(($1 + $2))
    echo "$result"
}
```

---

#### 7. Logs

**Fonction log basique** :
```bash
log()
{
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" >> "$LOG_FILE"
}
```

**Avec niveaux** :
```bash
log_info()  { echo "[INFO]  $*" >> "$LOG_FILE"; }
log_error() { echo "[ERROR] $*" >> "$LOG_FILE"; }
```

---

#### 8. Outils de vérification

**ShellCheck** :
```bash
shellcheck script.sh
```

**shfmt** :
```bash
shfmt -w script.sh
```

---

### Checklist qualité script

> [!tip] Avant de livrer un script

- [ ] ✅ **Shebang** présent (`#!/usr/bin/env bash`)
- [ ] ✅ **En-tête** documenté (utilité, usage, auteur)
- [ ] ✅ **Commentaires** sur logique complexe
- [ ] ✅ **Variables** entre guillemets (`"$var"`)
- [ ] ✅ **Fonctions** utilisées (pas de duplication)
- [ ] ✅ **Variables locales** dans fonctions (`local`)
- [ ] ✅ **Gestion d'erreurs** (vérifications, codes de sortie)
- [ ] ✅ **Pas de sudo** dans le script
- [ ] ✅ **Logs** pour traçabilité (optionnel)
- [ ] ✅ **Vérifié** avec ShellCheck
- [ ] ✅ **Formaté** avec shfmt
- [ ] ✅ **Testé** dans différents scénarios

---

## Glossaire technique

> [!note] Définitions essentielles pour le TSSR

| Terme | Définition |
|-------|------------|
| **Fonction** | Bloc de code nommé réutilisable |
| **Shebang** | Première ligne d'un script indiquant l'interpréteur (`#!/bin/bash`) |
| **Paramètre** | Argument passé à une fonction ou un script (`$1`, `$2`, etc.) |
| **`$#`** | Nombre d'arguments passés |
| **`$@`** | Tous les arguments passés |
| **`$?`** | Code de sortie de la dernière commande |
| **`return`** | Retourne un code de sortie (0-255) depuis une fonction |
| **`echo`** | Affiche du texte (peut servir de retour de fonction) |
| **`local`** | Déclare une variable locale à une fonction |
| **Portée (Scope)** | Zone où une variable est accessible |
| **Variable globale** | Variable accessible dans tout le script |
| **Variable locale** | Variable accessible uniquement dans la fonction |
| **Code de sortie** | Valeur numérique indiquant succès (0) ou erreur (1-255) |
| **`exit`** | Termine le script avec un code de sortie |
| **`&&`** | Opérateur logique ET (exécute si succès) |
| **`||`** | Opérateur logique OU (exécute si échec) |
| **Redirection `>&2`** | Redirige vers stderr (erreur standard) |
| **Redirection `>>`** | Ajoute à un fichier (append) |
| **Redirection `2>/dev/null`** | Ignore les erreurs |
| **`tee`** | Affiche ET écrit dans un fichier |
| **`$EUID`** | Effective User ID (0 = root) |
| **Log** | Fichier de journalisation des événements |
| **ShellCheck** | Outil d'analyse statique pour Bash |
| **shfmt** | Outil de formatage pour Bash |
| **Linter** | Outil d'analyse de code (détection erreurs, style) |
| **Formatter** | Outil de formatage automatique du code |

---

**Document créé le** : 21 novembre 2025  
**Version** : 1.0  
**Source** : Cours "Les scripts Bash - Partie 3" - Formation TSSR

> [!success] ✅ BON COURAGE POUR VOTRE TITRE RNCP TSSR !

> [!quote] Citation finale
> "Beaucoup de notions → Beaucoup de pratique
> 
> **Écrivez plein de scripts pour tout !!**"

---
