
**Durée : 45 min max | À refaire 3x cette semaine**  
**VM : SRVLX01 (Debian) | Accès via SSH depuis CLIENT01**  
**Basé sur : Fiche CRYPTO + Dossier Professionnel**

---

> ⚠️ **AVANT DE COMMENCER**  
> - SRVLX01 allumé + SSH fonctionnel  
> - Connecté en SSH : `ssh -p 22504 technicien@172.16.10.20`  
> - Travailler dans ton home : `cd ~`

---

## Exercice 1 — Hachage SHA256 (vérification d'intégrité)

**Contexte :** Dans ton DP tu écris *"je vérifie l'intégrité avec SHA256"*. Pratique-le ici.

**Q1.1** Crée un fichier `config.txt` avec du contenu, génère son empreinte SHA256, puis modifie-le et regénère l'empreinte. Explique ce que tu observes.

> [!success]- 🔓 Réponse
> ```bash
> # Créer le fichier
> echo "configuration serveur sweetcakes" > config.txt
> 
> # Générer l'empreinte
> sha256sum config.txt
> # → ex: a3f1c2... config.txt
> 
> # Modifier le fichier
> echo "modification" >> config.txt
> 
> # Regénérer l'empreinte
> sha256sum config.txt
> # → ex: 9b4d7e... config.txt  ← DIFFÉRENTE
> ```
> **Ce que tu dis à l'oral :**  
> "La moindre modification change totalement l'empreinte. C'est comme ça que je vérifie qu'un fichier téléchargé n'a pas été altéré."

---

**Q1.2** Compare deux empreintes : vérifie si `config.txt` correspond à une empreinte de référence.

> [!success]- 🔓 Réponse
> ```bash
> # Stocker l'empreinte dans un fichier référence
> sha256sum config.txt > config.sha256
> 
> # Vérifier automatiquement
> sha256sum -c config.sha256
> # → config.txt: OK  (si fichier intact)
> # → config.txt: FAILED  (si fichier modifié)
> ```

---

## Exercice 2 — Chiffrement Symétrique AES-256

**Contexte :** Dans ton DP tu écris *"chiffrant des fichiers en symétrique avec AES-256"*.

**Q2.1** Chiffre le fichier `config.txt` avec AES-256. Vérifie que le fichier chiffré est illisible. Puis déchiffre-le.

> [!success]- 🔓 Réponse
> ```bash
> # Chiffrer (il demande un mot de passe)
> openssl enc -aes-256-cbc -pbkdf2 -in config.txt -out config.enc
> 
> # Vérifier que c'est illisible
> cat config.enc
> # → caractères binaires illisibles ✓
> 
> # Déchiffrer
> openssl enc -d -aes-256-cbc -pbkdf2 -in config.enc -out config_dechiffre.txt
> cat config_dechiffre.txt
> # → configuration serveur sweetcakes ✓
> ```
> **Ce que tu dis à l'oral :**  
> "Symétrique = même clé pour chiffrer et déchiffrer. Rapide, utilisé pour les données volumineuses. Problème : comment partager la clé de façon sécurisée ?"

---

## Exercice 3 — Chiffrement Asymétrique RSA

**Contexte :** Dans ton DP tu écris *"en asymétrique avec RSA"*.

**Q3.1** Génère une paire de clés RSA (privée + publique). Identifie laquelle va sur le serveur et laquelle reste sur ton poste.

> [!success]- 🔓 Réponse
> ```bash
> # Générer la clé privée RSA 2048 bits
> openssl genrsa -out cle_privee.pem 2048
> 
> # Extraire la clé publique
> openssl rsa -in cle_privee.pem -pubout -out cle_publique.pem
> 
> # Voir la différence
> cat cle_privee.pem    # BEGIN RSA PRIVATE KEY
> cat cle_publique.pem  # BEGIN PUBLIC KEY
> ```
> - **Clé publique** → peut être partagée / mise sur le serveur  
> - **Clé privée** → NE QUITTE JAMAIS ton poste

---

**Q3.2** Chiffre `config.txt` avec la clé publique. Déchiffre-le avec la clé privée.

> [!success]- 🔓 Réponse
> ```bash
> # Chiffrer avec la clé PUBLIQUE
> openssl rsautl -encrypt -pubin -inkey cle_publique.pem \
>   -in config.txt -out config_rsa.enc
> 
> # Déchiffrer avec la clé PRIVÉE
> openssl rsautl -decrypt -inkey cle_privee.pem \
>   -in config_rsa.enc -out config_rsa_ok.txt
> 
> cat config_rsa_ok.txt
> # → configuration serveur sweetcakes ✓
> ```
> **Ce que tu dis à l'oral :**  
> "Asymétrique = clé publique pour chiffrer, clé privée pour déchiffrer. Plus lent que le symétrique mais pas besoin de partager de secret. C'est le principe utilisé dans SSH et HTTPS."

---

## Exercice 4 — Lien SSH / Asymétrique (ce que tu as déjà configuré)

**Contexte :** SSH utilise exactement le même principe RSA. Tu l'as déjà fait en Lab 1.

**Q4.1** Montre que ta clé SSH est bien une paire asymétrique. Où se trouve la clé publique sur le serveur ?

> [!success]- 🔓 Réponse
> ```bash
> # Sur le CLIENT (ta clé privée — ne quitte pas ton poste)
> cat ~/.ssh/id_ed25519       # PRIVATE KEY
> 
> # Sur le CLIENT (ta clé publique — copiée sur le serveur)
> cat ~/.ssh/id_ed25519.pub
> 
> # Sur le SERVEUR (dans authorized_keys)
> cat ~/.ssh/authorized_keys
> # → ta clé publique est là ✓
> ```
> **Ce que tu dis à l'oral :**  
> "SSH utilise le chiffrement asymétrique. Je génère la paire, je dépose la clé publique sur le serveur dans `authorized_keys`. Quand je me connecte, le serveur chiffre un défi avec ma clé publique, seul mon poste peut le déchiffrer avec la clé privée. Même si quelqu'un intercepte, sans la clé privée il ne peut rien faire."

---

## Exercice 5 — VPN : démonstration concept (Wireshark)

**Contexte :** Dans ton DP tu écris *"vérifié avec Wireshark que sans le VPN les données circulaient en clair, alors qu'avec le tunnel tout était chiffré"*.

**Q5.1** Sans VPN : génère du trafic HTTP non chiffré et capture-le avec tcpdump pour montrer que c'est lisible en clair.

> [!success]- 🔓 Réponse
> ```bash
> # Sur SRVLX01 : capturer le trafic HTTP
> sudo tcpdump -i any -A port 80 -w /tmp/capture_http.pcap &
> 
> # Générer du trafic HTTP (depuis CLIENT01 ou sur le serveur)
> curl http://172.16.10.20
> 
> # Arrêter la capture
> sudo pkill tcpdump
> 
> # Lire la capture : on voit le contenu en clair
> sudo tcpdump -r /tmp/capture_http.pcap -A | head -30
> # → GET / HTTP/1.1  (lisible !)
> # → Host: 172.16.10.20 (lisible !)
> ```
> **Ce que tu dis à l'oral :**  
> "Sans VPN, le trafic HTTP circule en clair sur le réseau. N'importe qui avec Wireshark peut lire les données. Un VPN chiffre tout le tunnel, rendant le trafic illisible à l'interception. C'est pour ça qu'un salarié distant utilise un VPN pour accéder au réseau de l'entreprise."

---

**Q5.2** Montre la différence avec du trafic HTTPS (chiffré).

> [!success]- 🔓 Réponse
> ```bash
> # Capturer du trafic HTTPS
> sudo tcpdump -i any -A port 443 -w /tmp/capture_https.pcap &
> 
> # Générer du trafic HTTPS
> curl https://google.com --insecure 2>/dev/null
> 
> sudo pkill tcpdump
> 
> # Lire la capture : données illisibles
> sudo tcpdump -r /tmp/capture_https.pcap -A | head -30
> # → caractères chiffrés illisibles ✓
> ```
> **Ce que tu dis à l'oral :**  
> "HTTPS = HTTP + TLS. TLS utilise la cryptographie hybride : asymétrique pour échanger la clé de session, puis symétrique pour chiffrer les données. C'est exactement ce que j'explique dans mon dossier professionnel."

---

## 📋 Récap — Ce que tu dois savoir dire à l'oral

| Commande | Ce que ça fait |
|----------|---------------|
| `sha256sum fichier` | Génère l'empreinte SHA256 (intégrité) |
| `sha256sum -c fichier.sha256` | Vérifie si le fichier est intact |
| `openssl enc -aes-256-cbc -pbkdf2 -in f -out f.enc` | Chiffre en symétrique AES-256 |
| `openssl enc -d -aes-256-cbc -pbkdf2 -in f.enc -out f` | Déchiffre AES-256 |
| `openssl genrsa -out cle.pem 2048` | Génère clé privée RSA |
| `openssl rsa -in cle.pem -pubout -out pub.pem` | Extrait la clé publique |
| `openssl rsautl -encrypt -pubin -inkey pub.pem` | Chiffre avec clé publique |
| `openssl rsautl -decrypt -inkey cle.pem` | Déchiffre avec clé privée |
| `tcpdump -i any -A port 80` | Capture trafic HTTP (en clair) |

---

## ⚠️ Piège classique à l'oral

> "La clé publique chiffre, la clé privée déchiffre → **pour la confidentialité**"  
> "La clé privée signe, la clé publique vérifie → **pour l'authentification (signature)**"  
> Ne pas confondre les deux modes !