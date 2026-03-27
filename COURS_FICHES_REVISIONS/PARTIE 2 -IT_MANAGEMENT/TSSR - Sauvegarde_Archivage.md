## ⚡ L'essentiel en 5 minutes - Sauvegarde & Archivage

### 📌 C'est quoi en 2 lignes ?
Processus de copie des données sur supports externes pour récupération en cas de panne (sauvegarde) ou conservation légale long terme (archivage). Essentiel pour garantir la continuité d'activité et respecter obligations réglementaires.

---

### 💡 Concepts clés à retenir :

* **Sauvegarde** : Copie de données sur support externe pour récupération rapide en cas d'incident
* **Archivage** : Conservation long terme de données non utilisées actuellement, avec suppression de la production
* **PRA (Plan Reprise Activité)** : Procédures pour restaurer l'activité après incident majeur
* **PCA (Plan Continuité Activité)** : Stratégie pour maintenir services critiques pendant incident
* **Restauration** : Opération de recopie des données sauvegardées vers production (complète ou partielle)
* **Catalogue** : Base de données référençant localisation précise de chaque fichier sauvegardé
* **Snapshot** : Copie instantanée état système/données à un instant T (sans arrêt service)
* **RPO (Recovery Point Objective)** : Perte de données acceptable = intervalle entre 2 sauvegardes
* **RTO (Recovery Time Objective)** : Temps acceptable pour restaurer service après incident

---

### 💻 Commandes essentielles :

```bash
# 🐧 Linux - tar (archivage/sauvegarde simple)
tar -czf backup.tar.gz /chemin/dossier       # Créer archive compressée
tar -xzf backup.tar.gz                       # Extraire archive
tar -tzf backup.tar.gz                       # Lister contenu sans extraire

# 🐧 Linux - rsync (synchronisation/sauvegarde incrémentale)
rsync -av /source/ /destination/             # Copie avec archivage des attributs
rsync -av --delete /source/ /destination/   # Synchronisation miroir
rsync -avz user@serveur:/source/ /backup/   # Sauvegarde distante SSH
rsync -av --link-dest=/backup/jour1 /source/ /backup/jour2  # Incrémentale

# 🐧 Linux - dd (clonage disque/partition)
dd if=/dev/sda of=/backup/disk.img bs=4M    # Cloner disque entier
dd if=/dev/sda1 of=/backup/partition.img    # Cloner partition
dd if=/backup/disk.img of=/dev/sdb bs=4M    # Restaurer image

# 🐧 Linux - Borg Backup (sauvegarde incrémentale dédupliquée)
borg init --encryption=repokey /chemin/repo                 # Initialiser dépôt
borg create /repo::backup-2025-02-12 /home /etc            # Créer sauvegarde
borg list /repo                                             # Lister sauvegardes
borg extract /repo::backup-2025-02-12                      # Restaurer
borg prune --keep-daily=7 --keep-weekly=4 /repo            # Nettoyer anciennes

# 🐧 Linux - LVM snapshot
lvcreate -L 5G -s -n snap_backup /dev/vg/lv_data           # Créer snapshot
mount /dev/vg/snap_backup /mnt/snapshot                     # Monter snapshot
rsync -av /mnt/snapshot/ /backup/                           # Sauvegarder depuis snapshot
umount /mnt/snapshot && lvremove /dev/vg/snap_backup       # Supprimer snapshot
```

```powershell
# 🪟 Windows - Robocopy (sauvegarde incrémentale)
robocopy C:\Source D:\Backup /MIR                          # Miroir (synchronisation)
robocopy C:\Source D:\Backup /E /DCOPY:T                   # Copie récursive
robocopy C:\Source \\serveur\backup /Z /LOG:backup.log     # Sauvegarde réseau avec log

# 🪟 Windows - wbadmin (Windows Server Backup)
wbadmin start backup -backupTarget:E: -include:C:          # Sauvegarde disque C vers E
wbadmin start systemstatebackup -backupTarget:E:           # Sauvegarde état système
wbadmin get versions                                        # Lister sauvegardes
wbadmin start recovery -version:XX -itemType:File          # Restaurer fichiers

# 🪟 Windows - Veeam / Backup Exec (via GUI généralement)
# Outils commerciaux avec interface graphique

# 🪟 Windows - Export VM Hyper-V
Export-VM -Name "VM01" -Path "D:\Backup"                   # Exporter VM complète
```

```bash
# 🌐 Cron - Planification automatique Linux
# Éditer crontab : crontab -e

# Sauvegarde quotidienne à 2h du matin
0 2 * * * /usr/local/bin/backup-script.sh

# Sauvegarde hebdomadaire dimanche 3h
0 3 * * 0 /usr/local/bin/backup-weekly.sh

# Sauvegarde mensuelle 1er du mois 4h
0 4 1 * * /usr/local/bin/backup-monthly.sh
```

---

### 📐 Stratégies de sauvegarde :

**Règle 3-2-1 (Best Practice) :**
- **3** copies des données (1 prod + 2 sauvegardes)
- **2** types de supports différents (disque + bande / cloud)
- **1** copie hors-site (protection sinistre)

**Schéma classique hebdomadaire :**
```
Dimanche    : Complète                    (base)
Lundi       : Incrémentale depuis Dim     (+ modifs lundi)
Mardi       : Incrémentale depuis Lun     (+ modifs mardi)
Mercredi    : Incrémentale depuis Mar     (+ modifs mercredi)
Jeudi       : Incrémentale depuis Mer     (+ modifs jeudi)
Vendredi    : Incrémentale depuis Jeu     (+ modifs vendredi)
Samedi      : Incrémentale depuis Ven     (+ modifs samedi)
Dimanche+1  : Complète (nouvelle base)
```

**Restauration complète nécessite :**
- Sauvegarde complète + TOUTES les incrémentales suivantes jusqu'à date cible

**Schéma différentielle :**
```
Dimanche    : Complète                    (base)
Lundi       : Différentielle depuis Dim   (modifs dim→lun)
Mardi       : Différentielle depuis Dim   (modifs dim→mar)
Mercredi    : Différentielle depuis Dim   (modifs dim→mer)
...
```

**Restauration différentielle nécessite :**
- Sauvegarde complète + DERNIÈRE différentielle seulement

---

### ⚠️ Pièges à éviter :

* ❌ **Ne jamais tester les restaurations** : Sauvegarde non testée = sauvegarde potentiellement inutile
* ❌ **Sauvegarder uniquement en local** : Incendie/inondation = perte prod ET sauvegardes
* ❌ **Sauvegardes sur même support que prod** : Panne disque = perte de tout
* ❌ **Négliger la sécurité des sauvegardes** : Vol/accès non autorisé aux sauvegardes = fuite données
* ❌ **Pas de chiffrement des sauvegardes cloud** : Données sensibles accessibles au prestataire
* ❌ **Oublier de vérifier l'état quotidien** : Erreur silencieuse pendant des semaines/mois
* ❌ **Conservation indéfinie** : Coût stockage explosif + données obsolètes inutiles
* ❌ **Sauvegarde pendant heures pleines** : Impact négatif sur production
* ❌ **Pas de catalogue** : Impossible de retrouver fichier spécifique dans vieille sauvegarde
* ❌ **Perdre les clés de chiffrement** : Données chiffrées = irrécupérables définitivement

---

### ✅ Bonnes pratiques :

* ✅ **Automatiser les sauvegardes** : Planification via cron/tâches planifiées pour fiabilité
* ✅ **Tester restaurations régulièrement** : Valider intégrité + s'entraîner aux procédures (trimestriel minimum)
* ✅ **Appliquer règle 3-2-1** : Protection optimale contre tous types sinistres
* ✅ **Chiffrer sauvegardes externalisées** : Protection confidentialité (cloud, bandes déportées)
* ✅ **Monitorer logs de sauvegarde** : Détection rapide des échecs
* ✅ **Documenter procédures** : Restauration rapide même par personnes non habituelles
* ✅ **Sauvegardes hors-ligne (air gap)** : Protection contre ransomware
* ✅ **Planifier la nuit/weekend** : Minimiser impact sur production
* ✅ **Définir RPO/RTO par service** : Adapter fréquence à criticité métier
* ✅ **Vérifier intégrité (checksums)** : Garantir données non corrompues

---

### 📚 Vocabulaire technique :

| Terme | Définition courte |
|-------|------------------|
| **Sauvegarde complète** | Copie intégrale de toutes les données (long, lourd, restauration simple) |
| **Sauvegarde incrémentale** | Uniquement modifs depuis dernière sauvegarde (rapide, léger, restauration complexe) |
| **Sauvegarde différentielle** | Uniquement modifs depuis dernière complète (compromis) |
| **Clonage** | Image bit-à-bit complète d'un disque/système (bare-metal restore) |
| **Snapshot** | Photo instantanée état système/données sans arrêt service |
| **Catalogue** | Base indexant localisation précise chaque fichier sauvegardé |
| **Péremption** | Durée conservation sauvegarde avant suppression/écrasement |
| **LTO (Linear Tape-Open)** | Standard bandes magnétiques haute capacité pour sauvegarde |
| **Déduplication** | Élimination données dupliquées pour économiser stockage |
| **Compression** | Réduction taille données via algorithme (gzip, bzip2, zstd) |
| **Air gap** | Sauvegarde déconnectée réseau (protection ransomware) |
| **Bare-metal restore** | Restauration complète sur matériel vierge (OS + données) |
| **Hot backup** | Sauvegarde à chaud (service actif) |
| **Cold backup** | Sauvegarde à froid (service arrêté) |
| **RGPD** | Règlement européen protection données personnelles |

---

### 🎯 À retenir ABSOLUMENT :

1. 💡 **Théorique** : Règle 3-2-1 → 3 copies, 2 supports différents, 1 hors-site = protection optimale
2. 💻 **Pratique** : `rsync -av --delete /source/ /backup/` + cron automatique + test restauration mensuel
3. ⚠️ **Piège** : Sauvegarde jamais testée = potentiellement inutilisable le jour J (TESTER RÉGULIÈREMENT !)

---

### 🔄 Comparaison types de sauvegardes :

| Type | Vitesse backup | Espace stockage | Vitesse restauration | Complexité restauration |
|------|----------------|-----------------|----------------------|-------------------------|
| **Complète** | ⏱️⏱️⏱️⏱️⏱️ Très lent | 💾💾💾💾💾 Très lourd | ⚡⚡⚡⚡⚡ Très rapide | ✅ Simple (1 fichier) |
| **Différentielle** | ⏱️⏱️⏱️ Moyen | 💾💾💾 Moyen | ⚡⚡⚡ Moyen | ✅✅ Assez simple (complète + 1 diff) |
| **Incrémentale** | ⏱️ Très rapide | 💾 Très léger | ⚡ Lent | ❌❌ Complexe (complète + toutes incr) |

---

### 📊 Durées légales conservation (exemples France) :

| Type document | Durée conservation |
|---------------|-------------------|
| **Comptabilité** | 10 ans |
| **Contrats commerciaux** | 5 ans |
| **Factures** | 5 ans (10 ans TVA) |
| **Bulletins paie** | 5 ans |
| **Logs connexion** | 6 mois à 1 an (CNIL) |
| **Données RGPD** | Selon finalité (durée strictement nécessaire) |

**⚠️ Attention RGPD :**
- Conservation limitée à finalité
- Droit à l'oubli
- Sécurité des archives
- Sanctions lourdes si non-respect

---

### 💾 Supports de sauvegarde - Comparaison :

| Support | Capacité | Coût | Durabilité | Usage |
|---------|----------|------|------------|-------|
| **Disque dur interne** | 1-20 To | €€ | 3-5 ans | Sauvegarde fréquente |
| **Disque dur externe** | 1-20 To | €€ | 3-5 ans | Sauvegarde déportable |
| **SSD** | 250 Go-8 To | €€€€ | 5-10 ans | Sauvegarde rapide/critique |
| **Bande LTO-9** | 18 To (45 To compressé) | €€€ | 30+ ans | Archivage long terme |
| **Cloud S3/Glacier** | Illimité | €/To/mois | ∞ | Sauvegarde hors-site |
| **NAS** | 4-100+ To | €€€ | 5-7 ans | Sauvegarde centralisée |

---

### 🛠️ Outils de sauvegarde populaires :

**Open Source / Libre :**
- **Bacula / Bareos** : Solution entreprise complète (client-serveur)
- **Amanda** : Advanced Maryland Automatic Network Disk Archiver
- **Borg Backup** : Déduplication, chiffrement, compression
- **Restic** : Moderne, rapide, dédupliqué
- **rsync** : Simple, efficace, universel
- **Clonezilla** : Clonage disque/partition (bare-metal)
- **Duplicati** : Interface web, chiffrement, cloud

**Propriétaire :**
- **Veeam Backup & Replication** : Standard virtualisation
- **Acronis Cyber Backup** : Complet, cloud intégré
- **Veritas Backup Exec** : Entreprise
- **Commvault** : Grande échelle
- **Windows Server Backup** : Intégré Windows Server

---

### 🔐 Sécurité des sauvegardes :

```bash
# Chiffrement GPG avant envoi cloud
tar -czf - /data | gpg -e -r admin@entreprise.com > backup.tar.gz.gpg

# Chiffrement rsync via SSH
rsync -avz -e ssh /data/ user@backup-srv:/backups/

# Borg avec chiffrement intégré
borg init --encryption=repokey-blake2 /chemin/repo
# Mot de passe + clé = double protection

# Vérifier intégrité avec checksums
sha256sum backup.tar.gz > backup.tar.gz.sha256
sha256sum -c backup.tar.gz.sha256  # Vérifier intégrité
```

---

### ⏱️ Calcul RPO/RTO :

**RPO (Recovery Point Objective) :**
- Perte données acceptable
- Si sauvegarde quotidienne → RPO = 24h
- Si sauvegarde toutes les 6h → RPO = 6h
- Service critique → RPO faible (ex: 1h ou moins)

**RTO (Recovery Time Objective) :**
- Temps max acceptable pour restaurer
- Dépend de : taille données, vitesse réseau, procédures
- Service critique → RTO faible (ex: 4h)
- Service secondaire → RTO plus souple (ex: 48h)

**Exemple :**
```
Service : ERP comptabilité
Criticité : Haute
RPO cible : 4 heures (perte max acceptable)
→ Solution : Sauvegarde incrémentale toutes les 4h

RTO cible : 2 heures (max downtime acceptable)
→ Solution : Sauvegarde locale + procédure testée + équipe formée
```

---

### 📋 Checklist politique de sauvegarde :

- [ ] Définir données à sauvegarder (priorités métier)
- [ ] Choisir type sauvegarde (complète/incr/diff)
- [ ] Définir fréquence selon RPO
- [ ] Planifier créneaux (nuit/weekend)
- [ ] Définir péremption (7j/1m/1an...)
- [ ] Appliquer règle 3-2-1
- [ ] Chiffrer sauvegardes externalisées
- [ ] Automatiser via cron/planificateur
- [ ] Monitorer logs quotidiennement
- [ ] Tester restauration mensuellement
- [ ] Documenter procédures (wiki/doc)
- [ ] Former équipe aux restaurations
- [ ] Vérifier conformité légale (RGPD, etc)
- [ ] Auditer annuellement

---

### 🚨 Cas d'usage restauration :

**Scénario 1 : Fichier supprimé par erreur**
```bash
# Restauration partielle depuis sauvegarde
borg extract /repo::backup-hier /chemin/fichier.txt
# ou
rsync /backup/hier/chemin/fichier.txt /production/
```

**Scénario 2 : Crash serveur complet**
```bash
# 1. Réinstaller OS sur nouveau matériel
# 2. Restaurer dernière complète
# 3. Restaurer toutes incrémentales dans l'ordre
# 4. Reconfigurer réseau/services
# 5. Valider + remettre en prod
```

**Scénario 3 : Ransomware**
```bash
# 1. Isoler système infecté (déconnexion réseau)
# 2. Identifier dernière sauvegarde saine (avant infection)
# 3. Formater système
# 4. Réinstaller OS propre
# 5. Restaurer depuis sauvegarde HORS-LIGNE (air gap)
# 6. Mettre à jour sécurité avant reconnexion
```

**Scénario 4 : Test disaster recovery (DR drill)**
```bash
# Test annuel obligatoire
# 1. Simuler panne totale datacenter
# 2. Activer site secondaire / restaurer depuis backup
# 3. Chronométrer (vérifier RTO)
# 4. Vérifier intégrité données (vérifier RPO)
# 5. Documenter problèmes rencontrés
# 6. Améliorer procédures
```
