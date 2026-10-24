# 🗄️ Guide d'Installation et de Sécurisation de MariaDB sur Debian 12

Ce guide décrit la procédure étape par étape pour déployer, sécuriser et configurer le serveur de base de données **MariaDB** sur un système Debian 12 (Bookworm) vierge.

## 📋 Table des Matières
- [Mise à jour du Système](#mise-à-jour-du-système)
- [Installation de MariaDB](#installation-de-mariadb)
- [Vérification du Service Systemd](#vérification-du-service-systemd)
- [Sécurisation Initiale (mariadb-secure-installation)](#sécurisation-initiale-mariadb-secure-installation)
- [Configuration de la Base de Données et des Privilèges](#configuration-de-la-base-de-données-et-des-privilèges)

---

## 🔄 Mise à jour du Système

Avant d'installer tout nouveau paquet, il est indispensable de rafraîchir l'index des dépôts et de mettre à jour les paquets existants du système d'exploitation.

Exécutez la commande suivante avec les privilèges `root` ou `sudo` :
```bash
sudo apt update && sudo apt upgrade -y
```

---

## 📦 Installation de MariaDB

Debian 12 inclut les paquets officiels de MariaDB dans ses dépôts stables. Installez simultanément le serveur et le client natif :

```bash
sudo apt install mariadb-server mariadb-client -y
```

---

## ⚙️ Vérification du Service Systemd

À la fin de l'installation, le gestionnaire de services Systemd active et démarre automatiquement le processus MariaDB. Vérifiez son statut fonctionnel :

```bash
sudo systemctl status mariadb
```

### Commandes de gestion du cycle de vie du service :
* **Démarrer le service** : `sudo systemctl start mariadb`
* **Arrêter le service** : `sudo systemctl stop mariadb`
* **Activer le démarrage automatique au boot** : `sudo systemctl enable mariadb`

---

## 🔒 Sécurisation Initiale (mariadb-secure-installation)

MariaDB fournit un assistant interactif permettant d'éliminer les vulnérabilités par défaut (accès anonymes, bases de tests). Lancez l'outil de sécurisation :

```bash
sudo mariadb-secure-installation
```

### Réponses recommandées lors de l'exécution :
1. **Enter current password for root (enter for none):** Appuyez sur `Entrée` (aucun mot de passe par défaut).
2. **Switch to unix_socket authentication [Y/n]:** Saisissez `n` (conserve l'authentification standard pour l'administration globale).
3. **Change the root password? [Y/n]:** Saisissez `Y` et définissez un mot de passe robuste pour le compte administrateur root SQL.
4. **Remove anonymous users? [Y/n]:** Saisissez `Y` pour supprimer les accès invités.
5. **Disallow root login remotely? [Y/n]:** Saisissez `Y` pour interdire les connexions distantes avec le compte root.
6. **Remove test database and access to it? [Y/n]:** Saisissez `Y` pour nettoyer les bases temporaires.
7. **Reload privilege tables now? [Y/n]:** Saisissez `Y` pour appliquer immédiatement la nouvelle politique de sécurité.

---

## 🔑 Configuration de la Base de Données et des Privilèges

Pour des raisons de sécurité, une application ne doit jamais utiliser le compte `root`. Suivez cette procédure pour initialiser un environnement dédié avec des privilèges restreints.

### 1. Connexion à la CLI MariaDB
```bash
sudo mariadb -u root -p
```
*(Saisissez le mot de passe root SQL configuré à l'étape précédente).*

### 2. Requêtes SQL fondamentales
Exécutez les instructions suivantes au sein de l'invite MariaDB (`MariaDB [(none)]>`) :

```sql
-- Création de la base de données dédiée à l'application
CREATE DATABASE app_production_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Création de l'utilisateur applicatif local avec un mot de passe sécurisé
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'MotDePasseApplicatifTresSecurise123!';

-- Attribution des privilèges restreints uniquement sur la base dédiée
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER ON app_production_db.* TO 'app_user'@'localhost';

-- Rechargement des tables de droits pour appliquer les modifications
FLUSH PRIVILEGES;

-- Quitter l'interface en toute sécurité
EXIT;
```

### 3. Validation de la connexion restreinte
Vérifiez que le nouvel utilisateur accède correctement et exclusivement à sa base de données :
```bash
mariadb -u app_user -p -e "SHOW DATABASES;"
```
