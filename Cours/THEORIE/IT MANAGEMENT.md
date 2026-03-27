# 📝 QUIZ RÉVISION — INFRASTRUCTURE / SERVICES

---
---

## 📋 Suivi des incidents (ITIL)

1. **ITIL** =

> [!success]- 🔓 Réponse
> Méthode de **bonnes pratiques** pour gérer les services informatiques.

2. **Incident** =

> [!success]- 🔓 Réponse
> Événement imprévu qui **perturbe** le fonctionnement du SI (c'est l'**EFFET**).

3. **Problème** =

> [!success]- 🔓 Réponse
> La **cause** d'un ou plusieurs incidents (c'est la **CAUSE**).

4. **Helpdesk** =

> [!success]- 🔓 Réponse
> Équipe qui **reçoit et traite** les demandes des utilisateurs.

5. **SLA** =

> [!success]- 🔓 Réponse
> Service Level Agreement — contrat qui définit les **délais** et **niveaux de service**.

6. **CMDB** =

> [!success]- 🔓 Réponse
> Base de données qui recense tous les **équipements** et leur configuration.

---

## 🔄 Incident vs Problème (IMPORTANT)

| | Incident | Problème |
|--|----------|----------|
| **C'est** | | |
| **Exemple** | | |
| **Objectif** | | |

> [!success]- 🔓 Réponse
>
> | | Incident | Problème |
> |--|----------|----------|
> | **C'est** | L'**EFFET** | La **CAUSE** |
> | **Exemple** | "Je ne peux pas me connecter" | Serveur DHCP en panne |
> | **Objectif** | Restaurer le service vite | Trouver et éliminer la cause |

---

## 📈 Les 8 étapes de gestion d'incident

1.
2.
3.
4.
5.
6.
7.
8.

> [!success]- 🔓 Réponse
> 1. **Identification** (détecter)
> 2. **Notification** (signaler)
> 3. **Enregistrement** (créer le ticket)
> 4. **Catégorisation + Priorisation**
> 5. **Diagnostic**
> 6. **Suivi (ou escalade)**
> 7. **Résolution** (et documentation)
> 8. **Clôture**

---

## 🏢 Niveaux de support

| Niveau | Rôle |
| ------ | ---- |
|        |      |
|        |      |
|        |      |
|        |      |

> [!success]- 🔓 Réponse
>
> | Niveau | Rôle |
> | ------ | ---- |
> | **N0** | Enregistrement, tri |
> | **N1** | Résolution simple (procédures) |
> | **N2** | Analyse + résolution complexe |
> | **N3** | Expertise technique poussée |

---

## 📊 Priorités

| Priorité | C'est quoi |
| -------- | ---------- |
|          |            |
|          |            |
|          |            |

> [!success]- 🔓 Réponse
>
> | Priorité | C'est quoi |
> | -------- | ---------- |
> | **Mineur** | Peu gênant, peut attendre |
> | **Majeur** | Gênant mais pas bloquant |
> | **Critique** | **Bloquant**, intervention immédiate |

---


**Différence incident / problème ? (CP4 Q1.1)**

> [!success]- 🔓 Réponse
> - **Incident** = l'EFFET → événement imprévu qui perturbe le SI
> - **Problème** = la CAUSE → cause inconnue d'un ou plusieurs incidents
> - Un problème peut être **latent** (pas encore causé d'incident)

---

**DIAGNOSTIC : Un utilisateur appelle pour dire qu'il ne peut plus imprimer. Quelles étapes par téléphone ? (CP4 Q1.3)**

> [!success]- 🔓 Réponse
> 1. Accueillir l'utilisateur et **créer le ticket**
> 2. **Questionner** pour comprendre le problème (reformuler)
> 3. **Diagnostiquer** (recueil infos → analyse → test hypothèses)
> 4. Proposer une **solution** ou un **contournement**
> 5. Si non résolu → **escalader** au niveau supérieur
> 6. **Documenter** et **clôturer** le ticket

---
---

## ☁️ Le Cloud Computing

**Que signifient IaaS, PaaS et SaaS ?**

> [!success]- 🔓 Réponse
> - **IaaS** = Infrastructure as a Service → ressources virtualisées (serveurs, stockage)
> - **PaaS** = Platform as a Service → environnement de développement/déploiement
> - **SaaS** = Software as a Service → logiciels disponibles via abonnement en ligne

---

**Quels sont les avantages du cloud ?**

> [!success]- 🔓 Réponse
> - Flexibilité et évolutivité des ressources
> - Réduction des coûts d'infrastructure
> - Collaboration améliorée et accès à distance

---

**Quels sont les inconvénients du cloud ?**

> [!success]- 🔓 Réponse
> - Questions de sécurité et confidentialité des données
> - Dépendance vis-à-vis du fournisseur
> - Complexités dans la gestion et l'intégration

---

**Différence entre scalabilité verticale et horizontale ?**

> [!success]- 🔓 Réponse
> - **Verticale** (Scale Up/Down) = augmenter la puissance d'**un seul serveur** (+ CPU, + RAM). Simple mais limité par le matériel.
> - **Horizontale** (Scale Out/In) = ajouter **plus de serveurs**. Plus flexible, meilleure tolérance aux pannes, mais plus complexe à gérer.

---

**C'est quoi l'élasticité dans le cloud ? Quelle différence avec la scalabilité ?**

> [!success]- 🔓 Réponse
> - **Scalabilité** = capacité à augmenter/diminuer les ressources (manuellement ou planifié)
> - **Élasticité** = ajustement **automatique** des ressources en fonction de la demande en temps réel
> - L'élasticité est une scalabilité **dynamique et automatisée**

---
---

## 💾 Sauvegarde et Archivage

**C'est quoi une sauvegarde ?**

> [!success]- 🔓 Réponse
> Copie des données sur un autre support pour les récupérer en cas d'incident.

---

**Différence sauvegarde complète, incrémentale, différentielle ?**

> [!success]- 🔓 Réponse
> - **Complète** = copie tout
> - **Incrémentale** = modifs depuis la **dernière sauvegarde** (rapide)
> - **Différentielle** = modifs depuis la **dernière complète** (compromis)

---

**C'est quoi la règle 3-2-1 ?**

> [!success]- 🔓 Réponse
> - **3** copies (prod + 2 sauvegardes)
> - **2** supports différents
> - **1** copie hors-site

---

**Différence PRA et PCA ?**

> [!success]- 🔓 Réponse
> - **PRA** (Plan de Reprise d'Activité) = reprendre APRÈS un sinistre
> - **PCA** (Plan de Continuité d'Activité) = maintenir PENDANT un sinistre

---

**Différence sauvegarde et archivage ?**

> [!success]- 🔓 Réponse
> - **Sauvegarde** = copie pour récupérer (données actives en prod)
> - **Archivage** = stockage long terme (données supprimées de la prod)

---

**DIAGNOSTIC : Une restauration échoue. Quelles vérifications ?**

> [!success]- 🔓 Réponse
> 1. Support de sauvegarde accessible ?
> 2. Fichiers de sauvegarde intègres (pas corrompus) ?
> 3. Espace disque suffisant pour restaurer ?
> 4. Droits d'accès corrects ?
> 5. Consulter le **catalogue** de sauvegarde

---

**Quels sont les 3 composants de Bareos et leurs rôles ?**

> [!success]- 🔓 Réponse
> - **bareos-dir** (Director) = orchestre les sauvegardes (planification, catalogue)
> - **bareos-sd** (Storage Daemon) = gère le stockage
> - **bareos-fd** (File Daemon) = agent client, envoie les données
> Ports TCP : 9101 (dir), 9102 (fd), 9103 (sd)

---
---

## 📋 Journalisation

**C'est quoi la journalisation ? À quoi ça sert pour un TSSR ?**

> [!success]- 🔓 Réponse
> C'est l'enregistrement des traces d'activité des applications et OS (= logs). Ça sert à comprendre les dysfonctionnements, analyser l'utilisation, et détecter des tentatives d'intrusion.

---

**Quel est le protocole standard de journalisation sous Linux ? Quels ports utilise-t-il ?**

> [!success]- 🔓 Réponse
> **Syslog** (RFC 5424). Ports : 514/UDP (non sécurisé), 514/TCP, et **6514/TCP** (syslog over TLS, sécurisé).

---

**Où se trouvent les logs sous Windows ? Comment y accéder ?**

> [!success]- 🔓 Réponse
> Fichiers dans `C:\Windows\System32\winevt\Logs`. On y accède via l'**Observateur d'événements** (Event Viewer) : commande `eventvwr` ou outils d'administration.

---

**Quelle commande Linux permet de suivre un log en temps réel ?**

> [!success]- 🔓 Réponse
> `tail -f /var/log/syslog` (ou tout autre fichier de log). Le `-f` = "follow", affiche les nouvelles lignes au fur et à mesure.

---

**Pourquoi faut-il utiliser TLS pour transmettre les logs syslog à un serveur distant ?**

> [!success]- 🔓 Réponse
> Parce que les logs transitent par le réseau et peuvent contenir des **informations sensibles** (authentifications, erreurs…). Le chiffrement TLS (port 6514/TCP) protège la confidentialité et l'intégrité des logs en transit. On utilise TCP plutôt qu'UDP pour garantir la livraison.

---
---

## 📡 VIRTUALISATION

**C'est quoi un hyperviseur ?**

> [!success]- 🔓 Réponse
> Couche logicielle qui permet à plusieurs VM de fonctionner simultanément sur une même machine physique.

---

**Différence entre hyperviseur Type 1 et Type 2 ?**

> [!success]- 🔓 Réponse
> - **Type 1** (bare metal) : s'installe directement sur le matériel → meilleures performances
> - **Type 2** (hébergé) : s'installe sur un OS existant → plus simple mais moins performant

---

**C'est quoi l'overhead en virtualisation ?**

> [!success]- 🔓 Réponse
> Les ressources (CPU, RAM) consommées par l'hyperviseur lui-même pour gérer la virtualisation = coût supplémentaire en plus des VM.

---
---

## 📧 La Messagerie

**C'est quoi les 4 agents de la messagerie ?**

- **MUA** (Mail User Agent) =
- **MSA** (Mail Submission Agent) =
- **MTA** (Mail Transfer Agent) =
- **MDA** (Mail Delivery Agent) =

> [!success]- 🔓 Réponse
> - **MUA** (Mail User Agent) = le client de messagerie (logiciel de l'utilisateur)
> - **MSA** (Mail Submission Agent) = vérifie le contenu du mail et le transmet au MTA
> - **MTA** (Mail Transfer Agent) = élément principal du serveur SMTP, transmet les mails d'un serveur à un autre
> - **MDA** (Mail Delivery Agent) = remet le mail dans la BAL du destinataire

---

**Différence entre POP3 et IMAP ?**

> [!success]- 🔓 Réponse
> - **POP3** = télécharge les mails et les supprime du serveur → un seul poste
> - **IMAP** = synchronise la BAL entre serveur et postes → multi-postes, mails restent sur le serveur

---

**Un utilisateur consulte ses mails sur son PC au bureau et sur son téléphone. POP ou IMAP ?**

> [!success]- 🔓 Réponse
> **IMAP** obligatoire. POP téléchargerait les mails sur le premier appareil et les supprimerait du serveur → le 2e appareil ne verrait rien.

---

**Citez les 4 agents de la messagerie dans l'ordre d'envoi.**

> [!success]- 🔓 Réponse
> **MUA** (client) → **MSA** (vérifie) → **MTA** (transmet) → **MDA** (livre dans la BAL)

---

**DIAGNOSTIC : Un utilisateur dit "je peux envoyer des mails mais pas en recevoir". Que vérifier ?**

> [!success]- 🔓 Réponse
> - Vérifier la config du serveur **entrant** (POP/IMAP), pas SMTP (l'envoi marche)
> - Vérifier le port (110 pour POP, 143 pour IMAP)
> - Vérifier l'enregistrement **MX** dans le DNS du domaine
> - Vérifier que la **BAL** n'est pas pleine

---
---

## 📧 LES SERVICES BUREAUTIQUES

**Qu'est-ce qu'un service bureautique ? Cite les 4 catégories principales.**

> [!success]- 🔓 Réponse
> Logiciel/application pour les tâches courantes de bureau, proposé par la **DSI**.
> 4 catégories : **Messagerie**, **Stockage/partage de fichiers**, **Suites bureautiques**, **Prise en main à distance**.

---

**C'est quoi SMB ? Et Samba ?**

> [!success]- 🔓 Réponse
> - **SMB** (Server Message Block) = protocole de partage de fichiers/imprimantes, principalement **Windows**
> - **Samba** = implémentation **open-source** de SMB pour **Linux/Unix** → interopérabilité Windows/Linux

---
---

## 🖥️ Suivi de parc informatique

**C'est quoi un parc informatique ?**

> [!success]- 🔓 Réponse
> Ensemble des **équipements et logiciels** d'une entreprise.

---

**Quels sont les 3 axes de gestion de parc ?**

> [!success]- 🔓 Réponse
> **Entretenir** (maintenir) + **Développer** (évoluer) + **Optimiser** (améliorer)

---

**C'est quoi GLPI ?**

> [!success]- 🔓 Réponse
> Logiciel libre pour gérer le **parc + helpdesk**. Gestion passive.

---

**C'est quoi un MDM ?**

> [!success]- 🔓 Réponse
> Outil pour gérer les **appareils mobiles** à distance. Gestion active.

---

**Différence GLPI / MDM ?**

> [!success]- 🔓 Réponse
> - GLPI = parc fixe, passif (inventaire)
> - MDM = mobiles, actif (actions à distance)

---
---

## 🔄 Gestion des mises à jour

**Qu'est-ce que WSUS ?**

> [!success]- 🔓 Réponse
> Rôle Windows Server pour gérer les mises à jour Microsoft de façon centralisée. Téléchargement unique, contrôle des MAJ, planification, rapports.