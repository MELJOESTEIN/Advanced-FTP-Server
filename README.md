# Advanced FTP Server

## Présentation

Ce projet a été réalisé dans le cadre du cours SN-NETWORK OPERATIONS 1 pour configurer un serveur FTP sécurisé utilisant vsftpd sur une distribution Linux (Ubuntu/Debian). L'objectif principal est de résoudre les défis de transfert de fichiers volumineux entre équipes internes en mettant en place une solution robuste et sécurisée. Le serveur FTP permet à différents utilisateurs (administrateur, équipe de développement, équipe QA, et client) d'accéder à des répertoires spécifiques avec des permissions bien définies, tout en utilisant le chiffrement SSL/TLS pour sécuriser les connexions.

## Objectifs

- **Installer et configurer un serveur FTP sécurisé avec vsftpd**, incluant :
  - Désactivation de l'accès anonyme.
  - Activation du mode chroot pour confiner les utilisateurs.
  - Configuration de répertoires virtuels.
  - Support SSL/TLS pour des connexions sécurisées.

- **Gérer les utilisateurs avec des droits spécifiques** :
  - Admin : Accès complet à tous les répertoires.
  - DevTeam : Accès lecture/écriture à /var/ftp/dev.
  - QaTeam : Accès lecture seule à /var/ftp/qa.
  - clientUser : Accès lecture seule à /var/ftp/client.

- **Configurer un client FTP (FileZilla)** pour tester les connexions et transferts.
- **Documenter le projet** sur un dépôt GitHub privé avec captures d'écran et explications.

## Contexte

L'organisation a identifié des difficultés pour transférer des fichiers volumineux entre ses équipes. Ce projet vise à améliorer l'infrastructure informatique en mettant en place un serveur FTP centralisé, garantissant une collaboration fluide et sécurisée.

## Étapes Réalisées

### Étape 1 : Installation et Configuration du Serveur FTP (vsftpd)

#### Installation de vsftpd :

Mise à jour des paquets et installation :
```bash
sudo apt update
sudo apt install vsftpd
```

Démarrage et activation du service :
```bash
sudo systemctl start vsftpd
sudo systemctl enable vsftpd
```

#### Configuration de vsftpd :

Modification du fichier /etc/vsftpd.conf :
```bash
sudo nano /etc/vsftpd.conf
```

Paramètres configurés :
```
anonymous_enable=NO
local_enable=YES
write_enable=YES
chroot_local_user=YES
allow_writeable_chroot=YES
ssl_enable=YES
allow_anon_ssl=NO
force_local_data_ssl=YES
force_local_logins_ssl=YES
ssl_tlsv1=YES
ssl_sslv2=NO
ssl_sslv3=NO
rsa_cert_file=/etc/ssl/certs/vsftpd.crt
rsa_private_key_file=/etc/ssl/private/vsftpd.key
```

#### Génération du certificat SSL/TLS :

Création d'un certificat auto-signé :
```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
-keyout /etc/ssl/private/vsftpd.key -out /etc/ssl/certs/vsftpd.crt
```

#### Création des répertoires virtuels :

Création des répertoires pour chaque groupe :
```bash
sudo mkdir -p /var/ftp/dev /var/ftp/qa /var/ftp/client
```

#### Redémarrage du service :
```bash
sudo systemctl restart vsftpd
```

*Capture d'écran : Configuration de vsftpd.conf*

### Étape 2 : Création et Gestion des Utilisateurs

#### Création des utilisateurs :

Création des comptes avec répertoires personnels :
```bash
sudo useradd -m -s /bin/bash ftpadmin
sudo useradd -m -s /bin/bash devteam
sudo useradd -m -s /bin/bash qateam
sudo useradd -m -s /bin/bash clientuser
```

Définition des mots de passe :
```bash
sudo passwd ftpadmin
sudo passwd devteam
sudo passwd qateam
sudo passwd clientuser
```

#### Configuration des permissions :

Répertoire principal /var/ftp :
```bash
sudo chown ftpadmin:ftpadmin /var/ftp
sudo chmod 755 /var/ftp
```

Répertoire /var/ftp/dev (lecture/écriture pour devteam) :
```bash
sudo chown devteam:devteam /var/ftp/dev
sudo chmod 770 /var/ftp/dev
```

Répertoire /var/ftp/qa (lecture seule pour qateam) :
```bash
sudo chown qateam:qateam /var/ftp/qa
sudo chmod 550 /var/ftp/qa
```

Répertoire /var/ftp/client (lecture seule pour clientuser) :
```bash
sudo chown clientuser:clientuser /var/ftp/client
sudo chmod 550 /var/ftp/client
```

#### Configuration des liens symboliques pour chroot :

Création des liens pour accéder aux répertoires virtuels :
```bash
sudo ln -sf /var/ftp/dev /home/devteam/dev
sudo ln -sf /var/ftp/qa /home/qateam/qa
sudo ln -sf /var/ftp/client /home/clientuser/client
```

*Capture d'écran : Permissions des répertoires*

### Étape 3 : Installation et Configuration de FileZilla

#### Installation de FileZilla :
```bash
sudo apt install filezilla
```

#### Configuration des profils de connexion :

1. Ouverture de FileZilla et accès à Fichier > Gestionnaire de sites.
2. Création d'un profil pour chaque utilisateur :
   - Hôte : localhost ou adresse IP du serveur.
   - Protocole : FTP.
   - Chiffrement : Exiger FTP explicite sur TLS.
   - Utilisateur : ftpadmin, devteam, qateam, ou clientuser.
   - Mot de passe : Mot de passe défini.

3. Acceptation du certificat auto-signé lors de la première connexion.

#### Test des connexions :

Vérification que chaque utilisateur peut se connecter et accéder à son répertoire spécifique.

*Capture d'écran : Configuration FileZilla*

### Étape 4 : Transfert de Fichiers

#### Création de fichiers de test :

Création des fichiers avec sudo pour contourner les restrictions de permission :
```bash
sudo bash -c 'echo "Test pour DevTeam" > /var/ftp/dev/test.txt'
sudo bash -c 'echo "Test pour QaTeam" > /var/ftp/qa/test.txt'
sudo bash -c 'echo "Test pour clientUser" > /var/ftp/client/test.txt'
```

#### Tests de transfert avec FileZilla :

- **Utilisateur devteam** :
  - Upload d'un fichier dans /dev (succès).
  - Téléchargement de test.txt (succès).
  - Tentative d'accès à /qa ou /client (échec).

- **Utilisateur qateam** :
  - Téléchargement de test.txt depuis /qa (succès).
  - Tentative d'upload dans /qa (échec).
  - Tentative d'accès à /dev ou /client (échec).

- **Utilisateur clientuser** :
  - Téléchargement de test.txt depuis /client (succès).
  - Tentative d'upload dans /client (échec).
  - Tentative d'accès à /dev ou /qa (échec).

- **Utilisateur ftpadmin** :
  - Accès à tous les répertoires et transferts (succès).

*Capture d'écran : Transferts FileZilla*

### Étape 5 : Documentation et Livraison

#### Création du dépôt GitHub :

1. Création d'un dépôt privé nommé Advanced-FTP-Server sur GitHub.
2. Initialisation avec ce fichier README.md.

#### Ajout des fichiers et captures d'écran :

Création d'un dossier screenshots/ pour stocker les captures :
- vsftpd_conf.png : Contenu de /etc/vsftpd.conf.
- permissions.png : Résultat de ls -ld /var/ftp/*.
- filezilla_config.png : Gestionnaire de sites FileZilla.
- file_transfers.png : Transferts réussis/échoués.

Ajout d'un script utilitaire pour corriger les permissions :
```bash
git add fix_ftp_permissions.sh
```

#### Commit et push :
```bash
git init
git add .
git commit -m "Initialisation du projet Advanced FTP Server"
git remote add origin <URL_DU_DÉPÔT>
git push -u origin main
```

#### Préparation de la présentation :

Création d'un PowerPoint avec :
- Introduction : Contexte et objectifs.
- Étapes : Installation, configuration, tests.
- Résultats : Captures d'écran des transferts.
- Conclusion : Résumé et leçons apprises.

Répétition pour respecter la limite de 10 minutes.

*Capture d'écran : Présentation PowerPoint*

## Résolution des Problèmes Rencontrés

Lors de la mise en œuvre, plusieurs problèmes ont été rencontrés et résolus :

### Erreur FileZilla : "550 Create directory operation failed" :

- **Cause** : Permissions incorrectes sur les répertoires et mauvaise configuration des liens symboliques.
- **Solution** :
  - Correction des permissions avec chown et chmod.
  - Recréation des liens symboliques :
    ```bash
    sudo ln -sf /var/ftp/dev /home/devteam/dev
    ```
  - Redémarrage de vsftpd :
    ```bash
    sudo systemctl restart vsftpd
    ```

### Erreur TLS : "Decode error (50)" :

- **Cause** : Certificat auto-signé non accepté par FileZilla.
- **Solution** :
  - Acceptation manuelle du certificat dans FileZilla.
  - Vérification des paramètres SSL dans /etc/vsftpd.conf.

### Erreur de permission lors de la création des fichiers de test :

- **Cause** : Utilisateur Linux sans droits d'écriture sur /var/ftp/*.
- **Solution** :
  - Utilisation de sudo pour créer les fichiers :
    ```bash
    sudo bash -c 'echo "Test pour DevTeam" > /var/ftp/dev/test.txt'
    ```

*Capture d'écran : Erreurs résolues*

## Conseils Pratiques

- **Sauvegarde** : Sauvegarde toujours les fichiers de configuration avant modification :
  ```bash
  sudo cp /etc/vsftpd.conf /etc/vsftpd.conf.bak
  ```

- **Commits réguliers** : Utilise des messages de commit clairs :
  ```bash
  git commit -m "Ajout de la configuration vsftpd"
  ```

- **Pare-feu** : Vérifie que le port 21 est ouvert :
  ```bash
  sudo ufw allow 21
  ```

- **Tests rigoureux** : Teste chaque utilisateur séparément pour confirmer les restrictions d'accès.
- **Présentation** : Prépare une version PDF de ta présentation pour la revue.

## Pistes Avancées

- **Journalisation** : Active la journalisation pour auditer les connexions :
  ```
  xferlog_enable=YES
  xferlog_file=/var/log/vsftpd.log
  ```

- **Script d'automatisation** : Utilise le script fix_ftp_permissions.sh pour automatiser la gestion des permissions (voir fix_ftp_permissions.sh).
- **Pare-feu avancé** : Configure iptables pour limiter les connexions FTP à des IP spécifiques.

## Conclusion

Ce projet a permis de maîtriser la configuration d'un serveur FTP sécurisé, la gestion des permissions sous Linux, et l'utilisation d'un client FTP. Les défis rencontrés (permissions, TLS, chroot) ont été surmontés grâce à une analyse méthodique et des ajustements précis. Le dépôt GitHub et la présentation PowerPoint sont prêts pour la revue, avec une documentation complète et des captures d'écran illustrant chaque étape.
