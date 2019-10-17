# Moodle
![Moodle](https://img.shields.io/badge/Moodle-%203.7.2-blue?colorB=f98012)
![Apache2](https://img.shields.io/badge/Apache2-2.4-blue.svg?colorB=557697)
![PHP](https://img.shields.io/badge/php-7.2.23--fpm--alpine3.9-blue.svg?colorB=8892BF)
![MariaDB](https://img.shields.io/badge/MariadDB-10.3-blue.svg?colorB=0085B0)
![Cron](https://img.shields.io/badge/Cron-php%3A7.2.23--fpm--stretch-yellow)

Crée rapidement un espace de travail local pour Moodle (Apache2, PHP-FPM et MariaDB). L'espace de travail local est créé et géré par Docker Compose

## Mise en place de l'environnement
1. Ce positionner dans le repertoire /opt
2. Cloner ce repository: ```git clone https://github.com/xrog2005/moodle.git```
3. Aller dans le répertoire moodle: ```cd moodle```
4. Cloner les sources moodle 3.7 stable: ```git clone -b MOODLE_37_STABLE https://github.com/moodle/moodle.git html```
5. copier  env-variable: ```cp env-variable .env```
6. Construire les containers: ```docker-compose build```
7. Démarrer l'environnement: ```docker-compose up -d```
8. Arrêt de l'environnement: ```docker-compose down```


## Structure du docker-compose

| Service | Type | Responsabilité| Contient | Configuration |
| :--- |:--- | :--- | :---| :---|
| **httpd** | Container | Service http | httpd:2.4.41-alpine | https://hub.docker.com/_/httpd |
| **cron** | Container | Tache Cron pour Moodle | Debian10, Cron |  |
| **moodledata** | Volume | Magasin de données Moodle  | Données générées par moodle |  |
| **php** | Container | Gestionnaire de processus pour PHP | Alpine linux 3.9, PHP-FPM | PHP modules et dépendance Moodle |
| **db** | Container | base de donnée  | ubuntu:bionic, Serveur MariaDB  | https://github.com/mariadb-corporation/mariadb-server-docker |


## Variables d'environment 
Le tableau suivant décrit les variables d’environnement dans [**.env**](.env):

| Variable | Default value | Use |
| :--- |:--- |:--- |
| **REPO_FOLDER** | html | Dossier pour les sources  |
| **DOCUMENT_ROOT** | /usr/local/apache2/htdocs | Point de montage a l'intérieure du container pour le volume **REPO_FOLDER** |
| **MY_TZ** | Europe/Paris | Fuseau horaire des Containers |
| **MYSQL_ROOT_PASSWORD** | password | Mot de passe root bdd |
| **MYSQL_PORT** | 5432 | Port exposé de bdd MariaDB  |
| **MYSQL_DATABASE** | moodle | Bdd pour moodle |
| **MYSQL_USER** | user | utilisateur bdd pour moodle |
| **MYSQL_PASSWORD** | password | Mot de passe bdd pour Moodle |
| **MYSQL_STORAGE** | /var/lib/mysql | Point de montage a l'intérieur du container pour persister les données dans un volume |
| **MYSQL_CONFIG** | /etc/mysql/conf.d | Point de montage a l'intérieur du container pour configurer le serveur MariaDB |
| **MYSQL_ENRTYPOINT** |  |Point de montage a l'intérieur du container pour la création de la bdd a partir d'un dump |
| **PHP_EXPOSE** | 9000 | Port pour exposer le container  |
| **PHP_WORK** | /usr/local/apache2/htdocs | Point de montage a l'intérieur du container pour volume **REPO_FOLDER** |
| **PHP_CONFIG** | /usr/local/etc/php/php.ini |Point de montage a l'intérieur du container pour configurer php |
| **APACHE_HOST_HTTP_PORT** | 80 | Port pour exposer le container |
| **APACHE_HOST_HTTPS_PORT** | 443 | Port pour exposer le container |
| **APACHE_CONF** | /usr/local/apache2/conf/httpd.conf |Point de montage a l'intérieur du container pour httpd.conf |
| **APACHE_VHOST** | /usr/local/apache2/conf/vhosts | Point de montage a l'intérieur du container pour le vhost |
| **APACHE_CONFD** | /usr/local/apache2/conf/conf.d |Point de montage a l'intérieur du container pour le volume conf.d |
| **APACHE_LOGS** | /usr/local/apache2/logs:z | Point de montage a l'intérieur du container pour le dossier logs apache2  |



## Gestion des services
1. Arrêt d'un service
``` bash
docker-compose stop
# docker-compose stop <service>
```
2. Démarrage d'un service
``` bash
docker-compose start
# docker-compose start <service>
```

2. Démarrage de l'environnement
``` bash
docker-compose up -d
```
3. Arrêt du project
``` bash
docker-compose down
# Avec les volumes
# docker-compose --volumes down
```
4. Logs
``` bash
docker-compose logs
#docker-compose logs <service>
```

5. Plusieurs commandes Docker
``` bash
# Liste des containers
docker ps
# Liste des images
docker images
# Entrer dans un container
 docker exec -it <container_name> bash
# Verifier les log d'un container
docker logs <container_name> -f
Correspond a une commande tail -f
# Effacement des images dangling
docker rmi $(docker images -f "dangling=true" -q)
