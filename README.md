# MSPR - Mise en oeuvre d'une intégration continue

## Configuration du serveur

### Pré-requis :
* Serveur assez puissant (déterminer une configuration minimale)
* OS : Ubuntu 20.04 lts

### Docker & Docker Compose :

* Installation de docker et docker-compose
```shell
sudo apt install docker-compose
```
* Ajouter son user au groupe docker
```shell
sudo usermod -aG docker $USER
```
* Redémarrer la session
```shell 
sudo /sbin/reboot
```
* Lancer le service Docker 
```shell
sudo service docker start
```

### Web Proxy utilisant Docker, NGINX et Let's Encrypt :
L’installation de Nginx sera faite à l’aide d’un outil permettant de simplifier la création et la mise en ligne de plusieurs containers docker à l’aide d’un seul proxy Nginx. Cet outil permettra également de mettre à jour automatiquement les certificats SSL avec Let’s Encrypt.

#### L’installation de déroule de la façon suivante :

Récupération du code source : 
```shell
git clone git@github.com:PaulAboulinc/docker-compose-letsencrypt-nginx-proxy-companion.git
```

Création du fichier ".env” avec le fichier générique : 
```shell
cp .env.sample .env
```
Modification de la variable `IP` du fichier `.env` pour indiquer l’adresse IP de la machine : 

Exécution du script :
```shell
 ./start.sh
```

Le script “./start.sh” fait les actions suivantes : 
* Récupérer les variables dans le fichier `.env`
* Création d'un network docker pour les reverse proxy nommé `webproxy`
* Télécharge la dernière version du reverse proxy nginx pour les containers docker ([source](https://raw.githubusercontent.com/jwilder/nginx-proxy/master/nginx.tmpl))
* Fait un pull des images docker
* Construction des dockers en utilisant docker-compose ([source](https://github.com/PaulAboulinc/docker-compose-letsencrypt-nginx-proxy-companion/blob/master/docker-compose.yml))

### Portainer :

### Jenkins :

### Sonarqube :

### Redmine :
