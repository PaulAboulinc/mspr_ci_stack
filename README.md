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

![Web Proxy environment](https://github.com/evertramos/images/raw/master/webproxy.jpg)

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

#### Le script “./start.sh” fait les actions suivantes : 
* Récupérer les variables dans le fichier `.env`
* Création d'un network docker pour les reverse proxy nommé `webproxy`
* Télécharge la dernière version du reverse proxy nginx pour les containers docker ([source](https://raw.githubusercontent.com/jwilder/nginx-proxy/master/nginx.tmpl))
* Fait un pull des images docker
* Construction des dockers en utilisant docker-compose ([source](https://github.com/PaulAboulinc/docker-compose-letsencrypt-nginx-proxy-companion/blob/master/docker-compose.yml))

### Portainer :

#### Portainer Agent
API permettant à portainer d'analyser l'environnement docker présent sur le serveur (Containers, volumes, networks...)
> **Installation :**
> ```shell
> docker run -d -p 9001:9001 --name portainer_agent --restart=always \
>            -v /var/run/docker.sock:/var/run/docker.sock \
>            -v /var/lib/docker/volumes:/var/lib/docker/volumes \
>            portainer/agent
> ``` 

#### Portainer CE
Interface graphique pour exploiter les informations mise à dispositon par l'API portainer agent mais aussi d'effectuer des actions sur l'environnement docker en place.

> **Installation :**
> * L'installation se fait à l'aide du ficher `docker-compose.yml` suivant
> ```yml
>  version: '3'
>  services:
>  portainer:
>  container_name: portainer
>  image: 'portainer/portainer-ce'
>  expose:
>    - 80
>  restart: always
>  environment:
>    VIRTUAL_PORT: 9000
>    VIRTUAL_HOST: portainer.nonstopintegration.ml
>    LETSENCRYPT_HOST: portainer.nonstopintegration.ml
>    LETSENCRYPT_EMAIL: nonstopintegration@gmail.com
>  networks:
>  default:
>    external:
>     name: webproxy
> ```
>  * Pour construire et lancer le "container" executer la commande :
>  ```shell
>  docker-compose up --build -d
>  ```

> **Configuration :**
>  * Connecter Portainer à l'environnement docker qui doit être gérer
>  * Choisir Agent comme mode de connexion puis :
>    * Donner un **Nom** à l'environnement
>    * Renseigner dans **Agent URL**, l'adresse IP public du serveur ainsi que le port du Portainer Agent (ex : 0.0.0.0:9001)
> 
> Voici un exemple :   
> ![](https://nsa40.casimages.com/img/2021/02/03/210203105543683341.png)

### Jenkins :

### Sonarqube :

### Redmine :
