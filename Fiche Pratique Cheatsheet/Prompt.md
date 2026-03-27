
Tu es mon assistant de révision pour le TSSR (Technicien Supérieur Systèmes et Réseaux - Titre RNCP).

Crée-moi une fiche de révision en Markdown (.md) compatible Obsidian sur le sujet : **[DNS]**

---

## Référentiel TSSR — CCP de référence :

### Activité 1 : Exploiter l'infrastructure et assurer le support
- CCP 1 : Support utilisateur (ITIL, GLPI, incidents, escalade, base de connaissances, prise en main à distance)
- CCP 2 : Serveurs Windows & Active Directory (comptes, GPO, droits, LDAP, journaux, sauvegardes, supervision)
- CCP 3 : Serveurs Linux (comptes, droits, services, journaux, supervision, open source)
- CCP 4 : Réseau IP (OSI, TCP/IP, adressage, diagnostic, supervision, Wi-Fi, schémas)

### Activité 2 : Maintenir, faire évoluer et sécuriser
- CCP 5 : Infrastructure virtualisée (hyperviseur, cloud, IaaS/SaaS/PaaS, messagerie, AD/GPO)
- CCP 6 : Scripts et automatisation (Bash, PowerShell, planification, gestion de versions)
- CCP 7 : Sécurité réseau (pare-feu, VPN, PKI, certificats, proxy, DMZ, VLAN, routage, chiffrement)
- CCP 8 : Sauvegardes & restaurations (PRA, PCA, planification, stockage)
- CCP 9 : Déploiement postes (WDS, WSUS, PXE, master, procédures)

---

## Format — modèle exact à respecter :

**Structure :**
- Titre H1 : `# [Sujet] - Cheatsheet TSSR`
- Mention du CCP concerné sous le titre : `> CCP X — [intitulé]`
- Sections en H2 (3 à 5 max)
- Chaque section = tableau Markdown 2 colonnes

**Modèle de tableau :**
| Action | Commande / Valeur |
|--------|-------------------|
| Description courte | `commande` ou valeur |

**Sections types (à adapter) :**
- Concepts clés / définitions
- Commandes principales ou étapes clés
- Fichiers importants (si pertinent)
- Points de vigilance / erreurs fréquentes

**Règles strictes :**
- Fiche COURTE : 60 à 80 lignes maximum
- Pas de sommaire, pas d'introduction, pas de bloc callout Obsidian
- Pas de phrases longues — une ligne = une info
- Commandes entre backticks `comme ça`
- Langue : français
- Contenu aligné sur les compétences du CCP correspondant

---

## Exemple de rendu attendu :

# Linux - Gestion des utilisateurs - Cheatsheet TSSR
> CCP 3 — Exploiter des serveurs Linux

## Création et suppression

| Action | Commande |
|--------|----------|
| Créer un utilisateur | `useradd nom_user` |
| Créer avec home directory | `useradd -m nom_user` |
| Supprimer avec home | `userdel -r nom_user` |

## Fichiers importants

| Fichier | Contenu |
|---------|---------|
| `/etc/passwd` | Liste des utilisateurs |
| `/etc/shadow` | Mots de passe chiffrés |