
# MSPR - Mise en oeuvre d'une intégration continue

## Configuration du serveur

### Sommaire :

* [Pré-requis](#pré-requis-)
* [Docker & Docker Compose](#docker--docker-compose-)
* [Web Proxy utilisant Docker, NGINX et Let's Encrypt](#web-proxy-utilisant-docker-nginx-et-lets-encrypt-)
* [Portainer](#portainer-)
  * [Portainer Agent](#portainer-agent)
  * [Portainer CE](#portainer-ce)
* [Sonarqube](#sonarqube-)
* [Redmine](#redmine-)
* [Jenkins](#jenkins-)

### Pré-requis :

* Serveur assez puissant (déterminer une configuration minimale)
* OS : Ubuntu 20.04 lts

### Docker & Docker Compose :

Docker nous permettra de conteneuriser nos applications et outils de façons simple et efficace, nous utiliserons également docker-compose pour dockeriser nos applications frontend et backend.

> **Installation :**
>
> * Installation de docker et docker-compose
>
> ```shell
> sudo apt install docker-compose
> ```
>
> * Ajouter son user au groupe docker
>
> ```shell
> sudo usermod -aG docker $USER
> ```
>
> * Redémarrer la session
>
> ```shell 
> sudo /sbin/reboot
> ```
>
> * Lancer le service Docker 
>
> ```shell
> sudo service docker start
> ```

### Web Proxy utilisant Docker, NGINX et Let's Encrypt :

L’installation de Nginx sera faite à l’aide d’un outil permettant de simplifier la création et la mise en ligne de plusieurs containers docker à l’aide d’un seul proxy Nginx. Cet outil permettra également de mettre à jour automatiquement les certificats SSL avec Let’s Encrypt.

> ![Web Proxy environment](https://github.com/evertramos/images/raw/master/webproxy.jpg)

> **Installation :**
>
> Récupération du code source : 
>
> ```shell
> git clone git@github.com:PaulAboulinc/docker-compose-letsencrypt-nginx-proxy-companion.git
> ```
>
> Création du fichier ".env” avec le fichier générique : 
>
> ```shell
> cp .env.sample .env
> ```
>
> Modification de la variable `IP` du fichier `.env` pour indiquer l’adresse IP de la machine : 
>
> Exécution du script :
>
> ```shell
> ./start.sh
> ```
>
> **Le script “./start.sh” fait les actions suivantes :**
>
> * Récupérer les variables dans le fichier `.env`
> * Création d'un network docker pour les reverse proxy nommé `webproxy`
> * Télécharge la dernière version du reverse proxy nginx pour les containers docker ([source](https://raw.githubusercontent.com/jwilder/nginx-proxy/master/nginx.tmpl))
> * Fait un pull des images docker
> * Construction des dockers en utilisant docker-compose ([source](https://github.com/PaulAboulinc/docker-compose-letsencrypt-nginx-proxy-companion/blob/master/docker-compose.yml))
>
> **Exemple d'utilisation :**
>
> * Créer un fichier "docker-compose.yml" sous le format suivant :
>
> ```yml
>  version: '3'
>  services:
>  <nom_service>: #Remplacer par le nom souhaité pour le service
>  container_name: <nom_container> #Remplacer par le nom souhaité pour le container
>  image: '<image_docker>' #Remplacer par l'image docker souhaité
>  expose:
>    - 80
>  restart: always
>  environment:
>    VIRTUAL_PORT: <port_container> #Remplacer par le port de l'application dans le container 
>    VIRTUAL_HOST: <host> #Remplacer par le/les nom de domaine souhaité
>    LETSENCRYPT_HOST: <host> #Remplacer par le/les nom de domaine souhaité
>    LETSENCRYPT_EMAIL: <email> #Remplacer par l'email utilisée par letsencrypt
>  networks:
>  default:
>    external:
>     name: webproxy
> ```
>
>  * Pour construire et lancer le "container" executer la commande :
>
>  ```shell
>  docker-compose up --build -d
>  ```

### Portainer :

#### Portainer Agent

API permettant à portainer d'analyser l'environnement docker présent sur le serveur (Containers, volumes, networks...)

> **Installation :**
>
> ```shell
> docker run -d -p 9001:9001 --name portainer_agent --restart=always \
>         -v /var/run/docker.sock:/var/run/docker.sock \
>         -v /var/lib/docker/volumes:/var/lib/docker/volumes \
>         portainer/agent
> ```

#### Portainer CE

Interface graphique pour exploiter les informations mise à dispositon par l'API portainer agent mais aussi d'effectuer des actions sur l'environnement docker en place.

> **Installation :**
>
> * L'installation se fait à l'aide du ficher `docker-compose.yml` suivant
>
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
>
>  * Pour construire et lancer le "container" executer la commande :
>
>  ```shell
>  docker-compose up --build -d
>  ```

> **Configuration :**
>
>  * Connecter Portainer à l'environnement docker qui doit être gérer
>  * Choisir Agent comme mode de connexion puis :
>    * Donner un **Nom** à l'environnement
>    * Renseigner dans **Agent URL**, l'adresse IP public du serveur ainsi que le port du Portainer Agent (ex : 0.0.0.0:9001)
>
> Voici un exemple :   
> ![](https://nsa40.casimages.com/img/2021/02/03/210203105543683341.png)

### Sonarqube :

Sonarqube nous permettra d'analyser et de mesurer la qualité du code, les potentiels failles de sécurité ainsi que les informations relatives aux tests unitaires.

> **Installation :**
>
> * L'installation se fait à l'aide du ficher `docker-compose.yml` suivant
>
>  ```yml
> version: '3'
> services:
>   sonarqube:
>     container_name: sonarqube
>     image: 'sonarqube:lts'
>     expose:
>       - 80
>     restart: always
>     environment:
>       VIRTUAL_PORT: 9000
>       VIRTUAL_HOST: sonarqube.nonstopintegration.ml
>       LETSENCRYPT_HOST: sonarqube.nonstopintegration.ml
>       LETSENCRYPT_EMAIL: nonstopintegration@gmail.com
> 
> networks:
>   default:
>     external:
>       name: webproxy
>  ```
>
>  * Pour construire et lancer le "container" executer la commande :
>
>  ```shell
>  docker-compose up --build -d
>  ```

### Redmine :

Redmine nous permettra de créer des tickets pour chaque tâche à réaliser. À l'aide de ces tickets, nous pourrons plus facilement nous répartir les tâches, suivre l'avancement de celles-ci ainsi que garder un suivi sur l'historique de réalisation de celles-ci.

> **Installation :**
>
> * L'installation se fait à l'aide du ficher `docker-compose.yml` suivant
>
>  ```yml
> version: '3'
> services:
>   redmine:
>     container_name: redmine
>     image: 'redmine:latest'
>     expose:
>       - 80
>     restart: always
>     environment:
>       VIRTUAL_PORT: 3000
>       VIRTUAL_HOST: redmine.nonstopintegration.ml
>       LETSENCRYPT_HOST: redmine.nonstopintegration.ml
>       LETSENCRYPT_EMAIL: nonstopintegration@gmail.com
> 
> networks:
>   default:
>     external:
>       name: webproxy
>  ```
>
>  * Pour construire et lancer le "container" executer la commande :
>
>  ```shell
>  docker-compose up --build -d
>  ```


### Jenkins :

Jenkins est l'outil qui nous permettra de gérer notre intégration continue ainsi que les déploiements sur les différentes environnements du projet.

> **Installation :**
>
> * Java est pré-requis pour le bon fonctionnement de Jenkins :
>
> ```shell
> sudo apt install openjdk-8-jdk
> ```
>
> * Ajout de Jenkins au dépôt du serveur : 
>
> ``` shell
> wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
> sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
> sudo apt update
> ```
>
> * Installation de Jenkins :
>
> ``` shell
> sudo apt install jenkins
> ```
>
> * Ajouter l'utilisateur `jenkins` au groupe `docker` pour utiliser docker avec jenkins
>
> ``` shell
>  sudo usermod -aG docker jenkins
> ```
>
> * Démarrer le service `Jenkins` :
>
> ``` shell
>  sudo service jenkins start
> ```
>
>  **Configuration :**
>
> Pour configurer votre installation, consultez Jenkins sur son port par défaut,  `8080`  en utilisant votre nom de domaine ou l'adresse IP de votre serveur :  `http://jenkins.nonstopintegration.ml:8080`
> Vous devriez voir apparaître l'écran  **Unlock Jenkins**  qui affichera l'emplacement du mot de passe initial :
>  ![](https://assets.digitalocean.com/articles/jenkins-install-ubuntu-1604/unlock-jenkins.png)
>  Dans la fenêtre du terminal, utilisez la commande  `cat`  pour afficher le mot de passe :
>
> ```bash
> sudo cat /var/lib/jenkins/secrets/initialAdminPassword
> ```
>
> Copiez le mot de passe alphanumérique composé de 32 caractères du terminal et collez-le dans le champ  > **Administrator password**, puis cliquez sur  **Continue**.
> Ensuite, sélectionné l'installation des plugins suggérés. Une fois cette installation terminée, vous pourrez créer votre utilisateur administrateur.
>
> * Configuration des plugins : 
>
>   * "Dashboard > Administration Jenkins > Gestion des plugins > Onglet “Disponible”
>
>   * Cocher les plugins suivants
>
>     | **Nom**                          | **Version** | **Commentaire**                                              |
>     | -------------------------------- | :---------- | ------------------------------------------------------------ |
>     | Email Extension Plugin           | 2.81        | Ce plugin remplace l'éditeur de messagerie de Jenkins. Il permet de configurer tous les aspects des notifications par e-mail: quand un e-mail est envoyé, qui doit le recevoir et ce que dit l'e-mail |
>     | JUnit Plugin                     | 1.48        | Permet la publication des résultats des tests au format JUnit. |
>     | Docker Compose Build Step Plugin | 1.0         | Plugin Docker Compose pour Jenkins                           |
>     | Docker pipeline                  | 1.25        | Créez et utilisez des conteneurs Docker à partir de pipelines. |
>     | Docker API Plugin                | 3.1.5.2     | Ce plugin fournit l'API docker-java pour d'autres plugins.   |
>     | GitHub Branch Source             | 2.9.4       | Projets multibranches et dossiers d'organisation de GitHub. Maintenu par CloudBees, Inc. |
>     | SonarQube Scanner for Jenkins    | 2.13        | Ce plugin permet une intégration facile de SonarQube, la plateforme open source pour l'inspection continue de la qualité du code. |
>
>   * Cliquez sur “installer sans redémarrer”
>
>  * Configuraton du Github Hook :
>    * Sur Jenkins
>      * Aller sur "Dashboard > Administrer Jenkins > Configurer le système > Onglet Github"
>      * Cliquez sur "Avancé..." et cochez l'option "Specify another hook URL for GitHub configuration"
>      * Indiquez le lien : http://jenkins.nonstopintegration.ml:8080/github-webhook/
>    * Sur Github :
>      * Aller sur "Settings > Webhook > add webhook > cliquez sur add webhook"
>      * Copier le lien "http://jenkins.nonstopintegration.ml:8080/github-webhook/" dans le champ "Payload URL"
>      
> * Configuration de SonarQube : 
> 
>   * Configurer le serveur SonarQube : 
>     * Aller sur   "Dashboard > Administrer Jenkins > Configurer le système > Onglet SonarQube servers"
>     * Cliquer sur "Ajouter une installation SonarQube"
>     * Indiquer le nom : SonarQube 
>     * Indiquer URL du serveur : 
>     * Indiquer Server authentication token : Ajouter le token récupéré sur sonarqube (token a créer dans "My account > Security > Token")
>     
>   * Confirgurer SonarQube Scanner
>     * Aller sur   "Dashboard > Administrer Jenkins > Configuration globale des outils > Onglet SonarQube Scanner"
>     * Indiquer le nom : SonarScanner
>     * Cocher "Install automatically"
>     * Cliquer sur "Ajouter un installateur"
>     * Sélectionner "SonarQube Scanner 4.6.0.2311"
>
> * Configuration des emails :
>    * Aller sur "Dashboard > Administrer Jenkins > Configurer le système > Onglet Jenkins Location"
>      * Remplir le champ "Adresse email de l'administrateur système", exemple : `NonStopIntegration <nonstopintegration@gmail.com>`
>    * Aller sur "Dashboard > Administrer Jenkins > Configurer le système > Onglet Extended E-mail Notification"
>      * SMTP server : smtp.gmail.com
>      * SMTP Port	: 465
>      * SMTP Username : nonstopintegration@gmail.com
>      * SMTP Password : Renseigner les logins
>      * Cocher "Use SSL"
>      * Default user e-mail suffix : @gmail.com
>    * Aller sur "Dashboard > Administrer Jenkins > Configurer le système > Onglet Notification par email"
>      * Serveur SMTP : smtp.gmail.com
>      * Suffixe par défaut des emails des utilisateurs : @gmail.com
>      * Cocher "Use SMTP Authentication"
>      * Nom d'utilisateur : nonstopintegration@gmail.com
>      * Mot de passe : Renseigner les logins
>      * Cocher "Use SSL"
>      * Port SMTP	: 465
>
> Jenkins est maintenant installé et configuré, il ne reste qu'à se connecter sur l'url suivante : `http://jenkins.nonstopintegration.ml:8080`
>
> #### Création et configuration Pipeline Multibranches
>
> > **NB** Veuillez vous assurer de bien avoir installer le plugin ''**GitHub Branch Source**'' avant de continuer.
>
> ##### Création
>
> Dashboard > Nouveau Item > Pipeline Multibranches
>
> Entrer un nom pour votre pipeline puis cliquez sur **OK**
>
> ##### Configuration
>
> - **Branch sources** : Choisir GitHub
>   - Cocher "Repository HTTPS URL"
> - **Behaviours** (Ajouter les "Behaviours suivant")
>   - Discover branches
>     - Strategy : All branches
>   - Discover pull requests from origin
>     - Strategy : The current pull request revision
>   - Discover pull requests from forks
>     - Strategy : The current pull request revision
>     - Trust : From users with Admin or Write permission
>   - Discover tags
> - **Build Configuration**
>   - Mode : by Jenkinsfile
>   - Script Path : Jenkinsfile
> - **Scan Repository Triggers**
>   - Cocher : Periodically if not otherwise run
>   - Interval : 5 minutes



