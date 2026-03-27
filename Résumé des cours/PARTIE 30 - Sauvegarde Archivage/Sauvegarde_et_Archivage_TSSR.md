# Sauvegarde & Archivage

## Document de révision TSSR - Titre RNCP

---

**Formation** : Technicien Supérieur Systèmes et Réseaux (TSSR)

**Sujet** : Sauvegarde et Archivage des données

**Date** : Février 2026

**Type** : Synthèse de cours complète

---

## 📋 Sommaire

1. [[#Introduction|Introduction]]
   - [[#Au service des utilisateurs|Au service des utilisateurs]]
   - [[#Des programmes immuables|Des programmes immuables]]
   - [[#Les données|Les données]]
   - [[#Impact des pannes|Impact des pannes]]
   - [[#Gestion des pannes|Gestion des pannes]]
2. [[#Sauvegarde|Sauvegarde]]
   - [[#Définition de la sauvegarde|Définition]]
   - [[#Les supports de sauvegardes|Les supports de sauvegardes]]
   - [[#Nombre de copies et stockage|Nombre de copies et stockage]]
   - [[#Types de sauvegardes|Types de sauvegardes]]
   - [[#Fréquence des sauvegardes|Fréquence des sauvegardes]]
   - [[#Péremption|Péremption]]
   - [[#Planification des sauvegardes|Planification des sauvegardes]]
   - [[#Restauration|Restauration]]
   - [[#Clonage|Clonage]]
3. [[#Archivage|Archivage]]
   - [[#Définition de l'archivage|Définition]]
   - [[#Contraintes particulières|Contraintes particulières]]
   - [[#Considérations légales|Considérations légales]]
4. [[#Politique de sauvegarde|Politique de sauvegarde]]
   - [[#Définition de la politique|Définition]]
   - [[#Vérifications|Vérifications]]
   - [[#Compression|Compression]]
   - [[#Confidentialité|Confidentialité]]
5. [[#Points clés à retenir|Points clés à retenir]]
6. [[#Glossaire technique|Glossaire technique]]

---

## Introduction

> [!abstract] Vue d'ensemble
> La sauvegarde et l'archivage sont des composantes essentielles de la gestion d'un système d'information. Elles permettent de garantir la continuité de l'activité en cas d'incident et de respecter les obligations légales de conservation des données. Ce cours aborde les concepts fondamentaux, les bonnes pratiques et les stratégies à mettre en œuvre pour protéger efficacement les données d'une organisation.

> [!question] Réflexion préliminaire
> À quand remonte la dernière sauvegarde de votre poste de travail ? Comment est-ce que vous l'avez fait ?

### Pourquoi étudier la sauvegarde et l'archivage ?

En tant que **TSSR**, tu dois être capable de mettre en place et de gérer des systèmes de sauvegarde fiables pour protéger les données critiques de l'entreprise. La perte de données peut avoir des conséquences catastrophiques sur l'activité d'une organisation : perte financière, arrêt de production, non-conformité légale, perte de confiance des clients.

---

## Au service des utilisateurs

> [!info] Les applications dans un SI
> Au sein d'un Système d'Information (SI), des applications servent les utilisateurs. Elles peuvent prendre différentes formes : applications autonomes, architecture client/serveur, systèmes pair à pair, etc.

Dans l'ensemble, on peut considérer une application comme composée de :
- **Des programmes** (du code source et exécutable)
- **Des données** (configuration, données utilisateurs, bases de données, etc.)

### Des programmes immuables

> [!note] Caractéristiques des programmes
> L'utilisation de l'application ne modifie pas le code source. Ce code change uniquement lors de l'installation et des mises à jour.

**Moyens d'obtention des programmes :**
- Téléchargement depuis le site de l'éditeur ou un dépôt officiel
- Supports d'installation physiques (CD, clé USB, DVD, etc.)

> [!success] Point important
> Si on a besoin de (ré)installer un programme à l'identique, il n'y a pas de difficulté majeure à obtenir le code en spécifiant le numéro de version. L'installation sera "vierge" et identique à toutes les autres instances.

### Les données

> [!important] Données critiques
> Les données sont l'élément le plus précieux et le plus vulnérable d'une application.

**L'installation du programme implique généralement :**
- **Configuration** : adaptation du programme au contexte particulier de l'entreprise
- **Création de données de configuration** : paramètres spécifiques à cette instance

**L'utilisation du programme génère :**
- Des données spécifiques à cette instance d'application
- Des informations souvent très précieuses pour l'activité de la structure

> [!note] Format des données
> Ces données peuvent prendre la forme de fichiers et/ou de bases de données.

---

## Impact des pannes

> [!warning] Les ennuis, ça arrive !
> Un certain nombre de pannes ou d'incidents peuvent engendrer la perte totale ou partielle des données.

### Comment rétablir l'activité (la production) ?

**Étapes de récupération après incident :**

1. **Réinstallation et reconfiguration** (à l'identique ?)
   - Obtenir le programme dans sa version exacte
   - Reconfigurer l'application
   - Résultat : application vierge à nouveau

2. **Récupération des données** depuis une copie
   - C'est là qu'intervient la **sauvegarde**

> [!tip] Mnémotechnique
> **RAR** : Réinstaller - Archiver - Restaurer (les trois R de la reprise d'activité)

---

## Gestion des pannes

> [!important] Anticiper les problèmes
> La prise en compte des pannes est une composante cruciale de la gestion d'un SI moderne.

### Politiques de gestion des incidents

La politique à mettre en œuvre pour minimiser l'impact des pannes comprend :

| Acronyme | Définition | Objectif |
|----------|------------|----------|
| **PRA** | **Plan de Reprise d'Activité** | Redémarrer l'activité après un sinistre majeur |
| **PCA** | **Plan de Continuité d'Activité** | Maintenir l'activité même en cas d'incident |

> [!info] Différence PRA/PCA
> - Le **PRA** intervient **après** un sinistre pour restaurer l'activité
> - Le **PCA** vise à **prévenir** l'interruption de l'activité grâce à des systèmes redondants et des procédures d'urgence

---

## Sauvegarde

### Définition de la sauvegarde

> [!quote] Définition officielle
> La **sauvegarde** est un moyen de récupérer les données en cas d'incidents les rendant inaccessibles. Elle consiste à créer des copies des données sur d'autres supports physiques.

---

## Les supports de sauvegardes

> [!info] Où sauvegarder ?
> Le choix du support de sauvegarde dépend de plusieurs critères : coût, capacité, vitesse, fiabilité, durée de vie.

### Classification des supports

**Sauvegardes sur disques :**
- Autres disques sur le même serveur
- Autre serveur (local ou distant)
- Autre site géographique

**Périphériques amovibles :**
- **Bandes magnétiques** (LTO - Linear Tape-Open)
- **Disques durs externes** (USB, eSATA)
- **Disques optiques** (CD, DVD, Blu-ray)

**Prestataire extérieur :**
- Solutions **cloud** (AWS, Azure, Google Cloud, OVH, etc.)

> [!example] Exemple de configuration hybride
> Une entreprise peut utiliser des disques locaux pour les sauvegardes quotidiennes rapides, des bandes magnétiques LTO pour les sauvegardes mensuelles à long terme, et une réplication cloud pour la protection hors-site.

---

## Nombre de copies et stockage

> [!important] Stratégie de protection multicouches
> Pour maximiser la sécurité des données, il est essentiel de multiplier les copies et de varier les emplacements de stockage.

### Principes de protection

**Copies hors-ligne :**
- Protection contre les **rançongiciels** (ransomware)
- Protection contre le **piratage** et les accès non autorisés

**Copies hors-site :**
- Protection en cas de **désastre** (incendie, inondation, catastrophe naturelle)
- Garantie de la continuité d'activité même en cas de destruction du site principal

### La règle du 3-2-1

> [!success] Règle du 3-2-1 : Standard de l'industrie
> Cette règle est une pratique courante et recommandée pour la sauvegarde des données critiques.

| Chiffre | Signification | Explication |
|---------|---------------|-------------|
| **3** | 3 copies | Production + 2 sauvegardes |
| **2** | 2 types de support | Ex : disque dur + bande magnétique |
| **1** | 1 copie hors-site | Stockage géographiquement distant |

> [!tip] Conseil pratique
> À adapter selon la **criticité** des données. Les données les plus sensibles peuvent nécessiter une règle 4-3-2 ou même plus !

> [!example] Exemple d'application de la règle 3-2-1
> Pour une base de données clients :
> - **Copie 1** : Serveur de production
> - **Copie 2** : Disque NAS local (sauvegarde automatique nocturne)
> - **Copie 3** : Bande LTO stockée hors-site + réplication cloud

---

## Types de sauvegardes

> [!info] Ne pas tout sauvegarder
> Il existe différentes stratégies de sauvegarde permettant d'optimiser le temps et l'espace de stockage nécessaires.

### Tableau comparatif des types de sauvegardes

| Type | Description | Avantages | Inconvénients | Usage typique |
|------|-------------|-----------|---------------|---------------|
| **Complète (Full)** | Duplication de toutes les données | - Restauration facile et rapide<br>- Sauvegarde autonome | - Long<br>- Très consommateur en espace de stockage | Hebdomadaire ou mensuelle |
| **Incrémentale (Incremental)** | Uniquement les modifications depuis la **sauvegarde précédente** (quelle qu'elle soit) | - Rapide<br>- Peu consommateur d'espace | - Restauration complexe<br>- Nécessite toutes les sauvegardes intermédiaires | Quotidienne |
| **Différentielle (Differential)** | Uniquement les modifications depuis la **dernière sauvegarde complète** | - Compromis temps/espace<br>- Restauration plus simple que l'incrémentale | - Plus volumineuse que l'incrémentale<br>- Plus longue que l'incrémentale | Quotidienne ou bihebdomadaire |

> [!example] Exemple de stratégie combinée
> **Lundi** : Sauvegarde complète (Full)
> **Mardi à Vendredi** : Sauvegardes incrémentales
> 
> Pour restaurer les données du vendredi, il faudra :
> 1. Restaurer la sauvegarde complète du lundi
> 2. Appliquer l'incrémentale du mardi
> 3. Appliquer l'incrémentale du mercredi
> 4. Appliquer l'incrémentale du jeudi
> 5. Appliquer l'incrémentale du vendredi

> [!warning] Piège à éviter
> Avec les sauvegardes incrémentales, la perte d'une seule sauvegarde intermédiaire peut rendre impossible la restauration complète. Il est donc crucial de vérifier régulièrement l'intégrité de toutes les sauvegardes.

---

## Fréquence des sauvegardes

> [!important] Quand sauvegarder ?
> La fréquence des sauvegardes est un élément critique de la stratégie de protection des données.

### Principes de base

**Objectif :**
- Avoir toujours des données **"fraîches"** et récentes
- Minimiser la quantité de données perdues entre la dernière sauvegarde et la panne (notion de **RPO - Recovery Point Objective**)

### Trouver le bon compromis

La fréquence est un **compromis** entre plusieurs contraintes :

| Critère | Impact |
|---------|--------|
| **Consommation en espace de stockage** | Plus on sauvegarde souvent, plus on stocke de données |
| **Temps nécessaire** | Les sauvegardes prennent du temps |
| **Impact sur la production** | Les sauvegardes peuvent ralentir le système |
| **Criticité des données** | Les données critiques nécessitent des sauvegardes plus fréquentes |

> [!success] Configuration classique recommandée
> **1 sauvegarde complète par semaine** + **1 sauvegarde incrémentale par jour**

> [!example] Exemples selon la criticité
> - **Données critiques** (base de données transactionnelle) : Incrémentale toutes les heures + Complète quotidienne
> - **Données importantes** (fichiers de travail) : Incrémentale quotidienne + Complète hebdomadaire
> - **Données peu critiques** (archives) : Complète mensuelle

> [!tip] Conseil pratique
> Définir le **RPO** (Recovery Point Objective) acceptable : quelle quantité de données l'entreprise peut-elle se permettre de perdre ? 1 heure ? 1 jour ? 1 semaine ?

---

## Péremption

> [!info] Récupérer de l'espace
> La péremption définit la durée de conservation d'une sauvegarde avant sa suppression ou son archivage.

### Durée minimum

> [!important] Règle de base
> **Minimum** : conserver une sauvegarde jusqu'à la prochaine sauvegarde complète réussie.

### Conservation longue durée

**Pourquoi conserver longtemps les sauvegardes ?**

En général, on conserve les sauvegardes longtemps pour pouvoir **revenir plus loin dans le temps** en cas de :
- **Compromission invisible** : intrusion détectée tardivement
- **Données supprimées par erreur** : découverte tardive de la suppression
- **Maladresse** : erreur humaine non détectée immédiatement
- **Corruption progressive** : dégradation lente des données

### Exemple de stratégie de rétention

> [!example] Politique de rétention classique
> | Fréquence | Durée de conservation |
> |-----------|----------------------|
> | **Quotidiennes** | 1 semaine |
> | **Hebdomadaires** | 1 à 2 mois |
> | **Mensuelles** | 1 à 2 ans |
> | **Annuelles** | 5 à 10 ans (selon obligations légales) |

> [!tip] Optimisation de l'espace
> Utiliser un système de **rotation** : GFS (Grandfather-Father-Son)
> - **Son** (Fils) : sauvegardes quotidiennes
> - **Father** (Père) : sauvegardes hebdomadaires
> - **Grandfather** (Grand-père) : sauvegardes mensuelles

---

## Planification des sauvegardes

> [!important] Quand lancer les sauvegardes ?
> Le timing des sauvegardes est crucial pour minimiser leur impact sur la production.

### Impact sur la production

**Conséquences des sauvegardes :**
- **Arrêt du service** (dans la plupart des cas, pour garantir la cohérence)
- **Copie volumineuse** : forte sollicitation des disques et du réseau
- **Ralentissement** du système en production

### Stratégie de planification

> [!success] Bonne pratique
> Planifier les sauvegardes dans les **creux d'utilisation** du système.

> [!example] Exemple classique
> Lancement des sauvegardes **la nuit** (entre 22h et 6h du matin) lorsque l'activité est minimale.

### Techniques de minimisation de l'impact

> [!tip] Solutions techniques avancées
> - **Snapshots** : captures instantanées de l'état du système
> - **Sauvegardes en ligne** (hot backup) : sauvegarde sans arrêt du service
> - **Réseau dédié** pour les sauvegardes et le stockage (SAN - Storage Area Network)
> - **Déduplication** : élimination des données redondantes
> - **Compression à la volée** : réduction de la bande passante nécessaire

> [!note] Les snapshots
> Un **snapshot** est une copie instantanée de l'état d'un système de fichiers ou d'une base de données à un instant T. Il permet de continuer les opérations normales pendant que la sauvegarde se fait à partir du snapshot.

---

## Restauration

> [!important] Récupérer les données
> La restauration est l'opération inverse de la sauvegarde : recopier les données sauvegardées vers l'environnement de production.

### Types de restauration

| Type | Description | Cas d'usage |
|------|-------------|-------------|
| **Restauration complète** | Récupération de l'ensemble des données | Crash système, panne matérielle, disaster recovery |
| **Restauration partielle** | Récupération d'un fichier ou dossier spécifique | Suppression accidentelle, corruption de fichier, besoin d'une version antérieure |

### Le catalogue de sauvegardes

> [!quote] Définition du catalogue
> Le **catalogue** est un index qui référence précisément sur quel support de sauvegarde se trouve chaque fichier, pour chaque date de sauvegarde.

**Informations contenues dans le catalogue :**
- Nom et emplacement des fichiers
- Date et heure de la sauvegarde
- Type de sauvegarde (complète, incrémentale, différentielle)
- Support de stockage utilisé
- Métadonnées (taille, permissions, propriétaire, etc.)

> [!warning] Piège à éviter
> Le catalogue lui-même doit être sauvegardé ! Sans catalogue, retrouver un fichier spécifique dans des téraoctets de sauvegardes devient quasi impossible.

> [!tip] Conseil pratique
> Tester régulièrement les procédures de restauration pour :
> - Vérifier l'intégrité des sauvegardes
> - Former les équipes
> - Évaluer le temps de restauration (RTO - Recovery Time Objective)
> - Détecter d'éventuels problèmes avant qu'un vrai sinistre ne survienne

---

## Clonage

> [!info] Image de machine complète
> Le clonage est une technique de sauvegarde qui capture l'intégralité d'un système.

### Avantages et inconvénients

**Problématique :**
La reprise après sinistre peut être **longue** si on doit passer par les étapes :
1. Installation du système d'exploitation
2. Installation des applications
3. Configuration
4. Restauration des données

**Solution : le clonage**

> [!quote] Définition du clonage
> Le **clonage** consiste à créer une image complète et exacte d'une machine (système d'exploitation, applications, configuration, données).

| Avantage | Inconvénient |
|----------|-------------|
| ✅ Redémarrage de la production **beaucoup plus rapide** | ❌ Taille des sauvegardes plus importante |
| ✅ Pas besoin de réinstaller et reconfigurer | ❌ Temps de sauvegarde plus long |
| ✅ Restauration "clé en main" | ❌ Nécessite souvent des outils spécialisés |

### Cas particulier : machines virtuelles

> [!success] Virtualisation et clonage
> Le clonage des **machines virtuelles** est beaucoup plus simple et rapide que celui des machines physiques.

**Avantages de la virtualisation pour la sauvegarde :**
- Snapshot instantané
- Clone complet en quelques minutes
- Portabilité entre hyperviseurs
- Tests de restauration simplifiés

> [!example] Outils de clonage
> - **Machines physiques** : Clonezilla, Norton Ghost, Acronis True Image
> - **Machines virtuelles** : Fonctionnalités natives des hyperviseurs (VMware, Hyper-V, Proxmox)

---

## Archivage

### Définition de l'archivage

> [!quote] Définition officielle
> L'**archivage** consiste à conserver des données dont on n'a pas besoin maintenant pour un usage potentiel ultérieur.

**Processus d'archivage :**
1. **Copier** les données vers un support d'archivage
2. **Supprimer** les données de la production (pour libérer de l'espace)

> [!info] Relation avec la sauvegarde
> L'archivage est une notion proche de la sauvegarde et peut être géré avec les mêmes outils. La différence principale réside dans l'**objectif** :
> - **Sauvegarde** : protection contre la perte, données restent en production
> - **Archivage** : conservation à long terme, données retirées de la production

---

## Contraintes particulières

> [!warning] Des problèmes spécifiques à l'archivage
> L'archivage long terme présente des défis techniques uniques.

### Principales contraintes

| Contrainte | Explication | Solution |
|------------|-------------|----------|
| **Supports physiques adaptés** | Les supports doivent résister au temps | Utiliser des supports à longue durée de vie (bandes LTO, disques optiques M-DISC) |
| **Obsolescence des formats** | Les formats de fichiers peuvent devenir illisibles | Migration régulière vers des formats standards et ouverts |
| **Garantie d'intégrité** | Vérifier que les données ne sont pas corrompues | Contrôles d'intégrité réguliers (checksum, hash) |
| **Indexation et recherche** | Retrouver un document dans des années d'archives | Système de catalogage et métadonnées riches |

> [!example] Exemple d'obsolescence
> Un document créé avec WordPerfect 5.1 en 1990 est aujourd'hui difficile à ouvrir. Pour l'archivage long terme, privilégier des formats standards comme PDF/A, ODF, ou des formats texte.

> [!tip] Conseil pratique
> Pour les archives critiques, envisager une **migration périodique** (tous les 5-10 ans) vers les nouveaux formats et supports pour éviter l'obsolescence.

---

## Considérations légales

> [!important] Contraintes légales
> Certaines données doivent **légalement** être conservées pour des durées variables selon leur nature.

### Durées légales de conservation

> [!note] Exemples de durées légales (Source : Ministère de l'Économie et des Finances)

| Type de données | Durée de conservation | Justification |
|-----------------|----------------------|---------------|
| **Données comptables** | **10 ans** | Code de commerce |
| **Contrats et factures** | **5 ans** | Code civil |
| **Journaux de connexion** | **6 mois à 1 an** | LCEN (Loi pour la Confiance dans l'Économie Numérique) |
| **Bulletins de paie** | **5 ans** | Code du travail |
| **Documents fiscaux** | **6 ans** | Livre des procédures fiscales |

> [!warning] RGPD et données personnelles
> Les données à caractère personnel sont soumises à des règles particulières définies par le **RGPD** (Règlement Général sur la Protection des Données).

### Principes du RGPD pour l'archivage

> [!info] Conformité au RGPD (Source : CNIL)
> - **Minimisation** : ne conserver que les données nécessaires
> - **Limitation de la conservation** : définir des durées précises
> - **Sécurité** : protéger les archives contre les accès non autorisés
> - **Traçabilité** : documenter les traitements d'archivage

> [!example] Exemple pratique
> Les données RH d'un ancien salarié peuvent être conservées :
> - **Actif** : pendant la durée du contrat
> - **Intermédiaire** : 5 ans après le départ (pour défense en justice)
> - **Définitif** : certaines données peuvent être archivées jusqu'à 50 ans pour les droits à la retraite

> [!tip] Conseil pratique TSSR
> Mettre en place un **tableau de gestion documentaire** qui spécifie pour chaque type de données :
> - La durée de conservation en production
> - La durée de conservation en archivage
> - La base légale de conservation
> - Les conditions de suppression

---

## Politique de sauvegarde

### Définition de la politique

> [!quote] Définition
> La **politique de sauvegarde** définit les règles de gestion des sauvegardes au sein d'une organisation.

**Intégration :**
- Partie intégrante du **PRA** (Plan de Reprise d'Activité)
- Partie intégrante du **PCA** (Plan de Continuité d'Activité)

### Éléments définis par la politique

La politique de sauvegarde **définit** :

| Élément | Description |
|---------|-------------|
| **Quelles données sauvegarder** | Inventaire des données critiques, importantes et non critiques |
| **Fréquence** | Rythme des sauvegardes complètes et incrémentales |
| **Planification** | Horaires et jours de sauvegarde |
| **Péremption** | Durée de conservation des différents types de sauvegardes |
| **Procédures de gestion** | Modes opératoires pour les sauvegardes et restaurations |
| **Responsabilités** | Qui fait quoi, qui valide, qui contrôle |
| **Support et stockage** | Types de supports utilisés et lieux de stockage |
| **Tests** | Fréquence et procédures de test de restauration |

> [!important] Document vivant
> La politique de sauvegarde doit être un document **vivant**, régulièrement mis à jour en fonction :
> - De l'évolution du SI
> - Des retours d'expérience
> - Des changements réglementaires
> - Des évolutions technologiques

---

## Vérifications

> [!warning] Ne pas attendre qu'il soit trop tard !
> Une sauvegarde non testée est une sauvegarde potentiellement inutilisable.

### Recommandations critiques

> [!important] Attention aux fausses sécurités
> Ne pas avoir **trop confiance** dans ses sauvegardes sans vérification régulière.

### Actions essentielles

> [!success] Bonnes pratiques de vérification

**1. Vérifier l'état des sauvegardes régulièrement**
- Contrôler les **logs** des logiciels de sauvegarde
- Vérifier l'**intégrité** des supports de stockage
- Surveiller les **alertes** et **erreurs**
- Contrôler l'**espace disponible** sur les supports

**2. S'entraîner à restaurer et réinstaller les applications**
- Effectuer des **tests de restauration** réguliers
- Documenter les **procédures** de restauration
- Former le **personnel** sur les procédures
- Chronométrer le temps de restauration (**RTO**)

> [!example] Planning de tests recommandé
> - **Restauration partielle** (quelques fichiers) : mensuelle
> - **Restauration complète** sur environnement de test : trimestrielle
> - **Test de disaster recovery** (simulation complète) : annuelle

> [!tip] Mnémotechnique
> **SVEA** : Sauvegarder - Vérifier - Entraîner - Auditer

> [!warning] Causes d'échec fréquentes
> - Supports défectueux non détectés
> - Catalogues corrompus ou perdus
> - Procédures obsolètes ou incomplètes
> - Personnel non formé
> - Durée de restauration sous-estimée
> - Incompatibilités matérielles ou logicielles

---

## Compression

> [!info] Économiser de l'espace
> La compression permet d'optimiser l'utilisation de l'espace de stockage des sauvegardes.

### Principe

> [!quote] Définition
> La **compression** utilise un algorithme pour réduire la taille des données sans perte d'information.

### Avantages et inconvénients

| Avantages | Inconvénients |
|-----------|---------------|
| ✅ Réduit significativement la consommation de stockage | ❌ Augmente légèrement l'utilisation du CPU |
| ✅ Réduit la bande passante réseau nécessaire | ❌ Augmente le temps de traitement |
| ✅ Permet de stocker plus de sauvegardes | ❌ Nécessite de décompresser pour restaurer |
| ✅ Réduit les coûts de stockage | ❌ Corruption possible de toute la sauvegarde si l'archive est endommagée |

> [!note] Compression matérielle
> La compression est **parfois directement effectuée par le support**, notamment dans le cas des bandes magnétiques LTO qui intègrent une compression matérielle.

> [!example] Taux de compression typiques
> - Fichiers texte, logs : 80-90% de réduction
> - Bases de données : 50-70% de réduction
> - Images JPEG, vidéos : 0-10% (déjà compressés)
> - Fichiers systèmes : 40-60% de réduction

> [!tip] Conseil pratique
> Activer la compression pour la plupart des sauvegardes, mais faire des tests pour vérifier que l'overhead CPU est acceptable. Pour les systèmes critiques très sollicités, privilégier la déduplication à la compression.

---

## Confidentialité

> [!warning] Question de sécurité
> Les sauvegardes sont des copies de la production, elles doivent donc bénéficier des **mêmes contraintes de sécurité**.

### Principe de base

> [!important] Équivalence de sécurité
> Les sauvegardes contiennent les mêmes données sensibles que la production. Elles nécessitent donc le **même niveau de protection**.

### Cas de l'externalisation

> [!warning] Sauvegardes externalisées (notamment cloud)
> Lorsque les sauvegardes sont confiées à un prestataire externe ou stockées dans le cloud, le **chiffrement des données** devient **impératif**.

**Raisons du chiffrement :**
- Protection contre les accès non autorisés
- Conformité réglementaire (RGPD, HDS, etc.)
- Protection contre la compromission du prestataire
- Garantie de confidentialité durant le transport

### Gestion des clés de chiffrement

> [!danger] Attention à la gestion des clés !
> Une **bonne gestion des clés de chiffrement** est absolument critique.

**Principes essentiels :**
- Les clés doivent être **stockées séparément** des sauvegardes chiffrées
- Mettre en place un **système de sauvegarde des clés** (mais pas avec les mêmes sauvegardes !)
- Prévoir un **système de récupération** en cas de perte de clés
- **Documenter** les procédures d'utilisation des clés
- Limiter l'**accès** aux clés aux seules personnes autorisées

> [!warning] Piège mortel
> **Perdre la clé de chiffrement = perdre définitivement l'accès aux sauvegardes**. C'est l'équivalent de ne pas avoir de sauvegarde du tout !

> [!example] Bonnes pratiques de gestion des clés
> - Utiliser un **HSM** (Hardware Security Module) pour stocker les clés
> - Implémenter un système de **clés escrow** (dépôt de clés sécurisé)
> - Rotation régulière des clés de chiffrement
> - Procédure de récupération testée régulièrement

> [!tip] Conseil TSSR
> Pour les environnements critiques, envisager le **chiffrement bout-en-bout** où seul le client possède les clés de déchiffrement, même le prestataire de sauvegarde ne peut pas accéder aux données.

---

## En conclusion

> [!success] Les sauvegardes, c'est la base !
> Avoir une **politique de sauvegarde** rigoureuse et une **gestion quotidienne disciplinée** est indispensable pour tout technicien systèmes et réseaux.

### Principes fondamentaux à retenir

**Les trois piliers de la sauvegarde :**
1. **Planification** : définir une stratégie adaptée aux besoins
2. **Exécution** : automatiser et surveiller les sauvegardes
3. **Validation** : tester régulièrement les restaurations

### Outils disponibles

> [!info] De nombreux outils existent sur le marché

**Solutions libres :**
- **Bacula / Bareos** : solution enterprise complète
- **Amanda** : Advanced Maryland Automatic Network Disk Archiver
- **Borg Backup** : déduplication et chiffrement
- **Clonezilla** : clonage de disques et partitions
- **rsync / rsnapshot** : synchronisation incrémentale
- **Duplicati** : sauvegarde vers le cloud

**Solutions propriétaires :**
- Veeam Backup & Replication
- Acronis Cyber Backup
- Veritas NetBackup
- Commvault
- IBM Spectrum Protect

> [!tip] Choix de l'outil
> Le choix de l'outil dépend de :
> - La taille de l'infrastructure
> - Le budget disponible
> - Les compétences de l'équipe
> - Les besoins spécifiques (cloud, virtualisation, etc.)

---

## Points clés à retenir

> [!success] Synthèse pour le titre RNCP

### Concepts fondamentaux

- **Distinction programme/données** : Les programmes sont réinstallables, les données sont uniques et irremplaçables
- **Sauvegarde vs Archivage** : Protection vs Conservation long terme
- **PRA vs PCA** : Reprise après sinistre vs Continuité d'activité

### Types de sauvegardes

- **Complète (Full)** : Tout sauvegarder - Restauration simple mais longue et volumineuse
- **Incrémentale** : Modifications depuis dernière sauvegarde - Rapide mais restauration complexe
- **Différentielle** : Modifications depuis dernière complète - Compromis

### Stratégies essentielles

- **Règle du 3-2-1** : 3 copies, 2 supports différents, 1 hors-site
- **Fréquence classique** : 1 complète/semaine + 1 incrémentale/jour
- **Planification** : Sauvegardes nocturnes pour minimiser l'impact

### Bonnes pratiques

- **Tester régulièrement** les restaurations (ne jamais faire confiance aveuglément)
- **Automatiser** les sauvegardes et la surveillance
- **Documenter** toutes les procédures
- **Former** les équipes aux procédures de restauration

### Sécurité et conformité

- **Chiffrement obligatoire** pour les sauvegardes externalisées
- **Gestion des clés** critique (séparation clés/sauvegardes)
- **Conformité légale** : durées de conservation selon le type de données
- **RGPD** : minimisation et limitation de conservation des données personnelles

### Points de vigilance

- **Péremption** : conserver assez longtemps pour détecter les problèmes invisibles
- **Catalogue** : indispensable pour les restaurations partielles (et doit lui-même être sauvegardé)
- **Compression** : économie d'espace vs overhead CPU
- **Supports hors-ligne** : protection contre les ransomwares

---

## Glossaire technique

> [!note] Définitions essentielles pour le TSSR

| Terme | Définition |
|-------|------------|
| **Sauvegarde (Backup)** | Copie de données sur un support distinct pour permettre leur restauration en cas d'incident |
| **Archivage** | Conservation long terme de données retirées de la production, souvent pour des raisons légales |
| **PRA** | Plan de Reprise d'Activité - Stratégie de redémarrage après sinistre majeur |
| **PCA** | Plan de Continuité d'Activité - Mesures préventives pour maintenir l'activité |
| **RPO** | Recovery Point Objective - Perte de données maximale acceptable (ex : 24h) |
| **RTO** | Recovery Time Objective - Durée maximale acceptable d'interruption de service |
| **Sauvegarde complète** | Copie intégrale de toutes les données sélectionnées |
| **Sauvegarde incrémentale** | Copie uniquement des modifications depuis la dernière sauvegarde (quel que soit son type) |
| **Sauvegarde différentielle** | Copie uniquement des modifications depuis la dernière sauvegarde complète |
| **Snapshot** | Capture instantanée de l'état d'un système à un moment donné |
| **Clonage** | Image complète d'une machine (OS + applications + données + configuration) |
| **Restauration** | Opération de recopie des données sauvegardées vers la production |
| **Catalogue** | Index référençant l'emplacement de chaque fichier dans les sauvegardes |
| **Péremption** | Durée de conservation d'une sauvegarde avant suppression |
| **LTO** | Linear Tape-Open - Standard de bandes magnétiques pour sauvegarde |
| **NAS** | Network Attached Storage - Serveur de stockage réseau |
| **SAN** | Storage Area Network - Réseau dédié au stockage |
| **Déduplication** | Élimination des données redondantes pour économiser l'espace |
| **Chiffrement** | Transformation des données pour les rendre illisibles sans clé |
| **HSM** | Hardware Security Module - Module matériel sécurisé pour stocker les clés de chiffrement |
| **RGPD** | Règlement Général sur la Protection des Données - Cadre légal européen |
| **Rançongiciel (Ransomware)** | Logiciel malveillant qui chiffre les données et demande une rançon |
| **Règle 3-2-1** | Stratégie de sauvegarde : 3 copies, 2 supports, 1 hors-site |
| **GFS** | Grandfather-Father-Son - Stratégie de rotation des sauvegardes (quotidien-hebdo-mensuel) |
| **Cloud** | Stockage ou services informatiques externalisés accessibles via Internet |
| **Hot backup** | Sauvegarde à chaud, sans arrêt du service |
| **Cold backup** | Sauvegarde à froid, nécessitant l'arrêt du service |
| **WORM** | Write Once Read Many - Support non réinscriptible (protection contre suppression) |

---

> [!quote] Citation finale
> "Il y a deux types d'administrateurs : ceux qui font des sauvegardes, et ceux qui n'ont pas encore perdu de données." - Proverbe informatique

**Fin du document de révision**

---

**📚 Pour aller plus loin :**
- Documentation Bacula : https://www.bacula.org/documentation/
- Guide ANSSI sur la sauvegarde : https://www.ssi.gouv.fr/
- CNIL - Guide durée de conservation : https://www.cnil.fr/
