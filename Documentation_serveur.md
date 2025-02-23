# Documentation : Installation de Moodle sur Ubuntu Server (VM)

## 1. Préparation du serveur
### Mise à jour du système

```bash
sudo apt update && sudo apt upgrade -y
```

## 1.1. Installation des prérequis
### Installation d'Apache, MariaDB et PHP

```bash
sudo apt install apache2 mariadb-server php php-mysql php-xml php-mbstring php-curl php-zip php-gd unzip -y
```

## 1.3. Configuration de la base de données
### Sécurisation de MariaDB
Lancer la commande suivante et suivre les instructions :

```bash
sudo mysql_secure_installation
```

- **Enter current password for root ?** → Appuyer sur **Entrée**
- **Set root password ?** → **Yes**, puis définir un mot de passe sécurisé
- **Remove anonymous users ?** → **Yes**
- **Disallow root login remotely ?** → **Yes**
- **Remove test database ?** → **Yes**
- **Reload privilege tables now ?** → **Yes**

### Création de la base de données pour Moodle
Se connecter à MariaDB :

```bash
sudo mysql -u root -p
```

Créer la base et l'utilisateur Moodle :
```sql
CREATE DATABASE moodle DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'moodleuser'@'localhost' IDENTIFIED BY 'root';
GRANT ALL PRIVILEGES ON moodle.* TO 'moodleuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```
(Remplacer **root** par un mot de passe sécurisé.)

### Modifier un mot de passe MariaDB en cas d'erreur
Si une erreur a été commise lors de la création de l'utilisateur Moodle, changer le mot de passe avec :

```bash
sudo mysql -u root -p
```

Puis exécuter :
```sql
ALTER USER 'moodleuser'@'localhost' IDENTIFIED BY 'NouveauMotDePasse';
FLUSH PRIVILEGES;
EXIT;
```

Tester la connexion avec :
```bash
mysql -u moodleuser -p
```
(Saisir le nouveau mot de passe.)


# 2. Installation de Moodle sur le serveur
Une fois la base de données configurée, on pourra procéder à l'installation de Moodle sur le serveur.

## 2.1. Télécharger Moodle

On va récupérer la dernière version de Moodle :
```bash
cd /var/www/html
sudo wget https://download.moodle.org/download.php/direct/stable402/moodle-latest-402.tgz

```

--
Décompresse l’archive et place les fichiers au bon endroit :
```bash
sudo tar -xvzf moodle-latest-402.tgz
sudo mv moodle /var/www/html/
```


--
Attribuer les bons droits :
```bash
sudo chown -R www-data:www-data /var/www/html/moodle
sudo chmod -R 755 /var/www/html/moodle

```
--


## 2.2. Créer un dossier de données pour Moodle

Moodle a besoin d’un répertoire pour stocker ses fichiers :
```bash
sudo mkdir /var/www/moodledata
sudo chown -R www-data:www-data /var/www/moodledata
sudo chmod -R 755 /var/www/moodledata
```
--


## 2.3. Configurer Apache pour Moodle

On va créer un fichier de configuration pour Moodle :
```bash
sudo nano /etc/apache2/sites-available/moodle.conf
```
ou

```bash
sudo nano /etc/apache2/sites-available/000-default.conf
```

---

**moodle.conf**
```fichier conf
<VirtualHost *:80>
    ServerAdmin admin@monsite.com
    DocumentRoot /var/www/html/moodle
    ServerName monsite.com
    ServerAlias www.monsite.com

    <Directory /var/www/html/moodle>
        Options FollowSymlinks  #Permet de suivre les liens symboliques
        AllowOverride All       #Permet d'utiliser des fichiers .htaccess pour la configuration du répertoire
        Require all granted     #Permet à tout le monde d'accéder au répertoire
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
--
**Sauvegarder**

Active le site et redémarre Apache :
```bash
sudo a2ensite moodle
sudo systemctl reload apache2
```

On peut enfin accéder à l'interface d'installation de moodle sur http://ip_du_serveur:80 ou http://localhost:80 .