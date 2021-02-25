# Installation du poste de travail

## Prérequis

* Ubuntu 20;04 installé sur la machine
* Git
* Docker et Docker Compose
* IntelliJ

## Installation des prérequis

### Paquets de base à installer

Afin d'avoir des outils de base commun, nous allons installer par défaut les paquets ubuntu suivants : `git` `vim` `meld` `url` `wget` `ssh`

* Avant d'installer les paquets, faire une mise à jour et redémarrer le PC : 

```shell
sudo apt update && sudo apt upgrade && reboot
```

* Ensuite, voici la commande pour installer les paquets nécessaires : 

```shell
sudo apt install git vim meld curl wget ssh
```


### Outils JetBrains

Pour cela il faut installer `fuse` : https://github.com/AppImage/AppImageKit/wiki/FUSE

~~~shell
sudo apt-get install fuse
sudo modprobe fuse
sudo groupadd fuse

user="$(whoami)"
sudo usermod -a -G fuse $user
~~~

Télécharger Jetbrains ToolBox uniquement : https://www.jetbrains.com/toolbox/app/
Après cela et depuis la toolbox, télécharger l'IDE **IntelliJ**

### Installation docker et docker-composer

Docker nous permettra de conteneuriser nos applications et outils de façons simple et efficace, nous utiliserons également docker-compose pour dockeriser nos applications frontend et backend.

> * **Installation de docker et docker-compose**
>
> ```shell
> sudo apt install docker-compose
> ```
>
> * **Ajouter son user au groupe docker**
>
> ```shell
> sudo usermod -aG docker $USER
> ```
>
> * **Redémarrer la session**
>
> ```shell 
> sudo /sbin/reboot
> ```
>
> * **Lancer le service Docker** 
>
> ```shell
> sudo service docker start
> ```



## Installation et utilisation de l'environnement de développement back-end

* Pour l'environnement local, utiliser le fichier `docker-compose.yml`

```yml
version: '3'

services:
  app:
    build: .
    container_name: api_backend
    depends_on:
      - dbpostgres
    links:
      - dbpostgres
    volumes:
      -  ./:/home/app/
    environment:
      - SPRING_DATASOURCE_URL=jdbc:postgresql://dbpostgres:5432/cooking
      - SPRING_DATASOURCE_USERNAME=cooking
      - SPRING_DATASOURCE_PASSWORD=cooking
      - SPRING_JPA_HIBERNATE_DDL_AUTO=update
    ports:
      - 7001:8080

  dbpostgres:
    image: postgres:13.1-alpine
    container_name: dbpostgres
    volumes:
      - postgres:/var/lib/postgresql
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: cooking
      POSTGRES_USER: cooking
      POSTGRES_PASSWORD: cooking
    ports:
      - 5432:5432

volumes:
  postgres_data:
  postgres:
```

> Ce fichier **docker-compose** permet de :
>
> * Pour le service `backend`
>   * Indique que `dbpostgres` est pré-requis pour le construction du service `app`
>   * Construire un conteneur appelé `recipe_back_msp` à partir du **Dockerfile** présent à la racine du projet
>   * Lier au conteneur `dbpostgres`
>   * Utiliser la racine du répertoire local comme **volumes** et le lier au source du container
>   * Renseigner les variables d'environnements dont l'application à besoin
>   * Rediriger le port **8080** du container vers le **7001** de la machine parent
>  * Pour le service `db`
>    * Construire le container `dbpostgres` à partir de l'image `postgres:13.1-alpine`
>    * Renseigner les variables d'environnements nécessaires à la base de donnée
>    * Lier le dossier `/var/lib/postgresql` au volume `postgres`
>    * Lier le dossier `/var/lib/postgresql/data` au volume `postgres_data`
>    * Rediriger le port **5432** du container vers le **5432** du serveur
>  * Pour la partie `volumes`
>    * Création du volume `postgres`
>    * Création du volume `postgres_data`

* Pour construire le container et le déployer en local

```bash
docker-compose up --build -d
```

* L'Application est disponible à l'URL suivant : 

```html
http://localhost:7001/api/
```

> **NB :** Les sources local sont liées à celle présente dans le container, du coup pas besoin de build de nouveau à chaque changement dans le code.

### Dépendances

| Dépendance         | Version | Commentaire                                                  |
| ------------------ | ------- | ------------------------------------------------------------ |
| Spring Web         | 2.4.1   | Permet de build (utilise apache tomcat)                      |
| Spring Data JPA    | 2.4.1   | Utilise Hibernate pour gérer la persistance                  |
| PostgresSQL driver | 42.2.18 | Driver pour se connecter à la base de données PostgesSQL     |
| Springdoc openapi  | 1.5.1   | Permet de générer la documentation                           |
| Junit              | 4.13.1  | Permet de réaliser les tests unitaires                       |
| Jasper Reports     | 6.1.0   | Permet de générer des pdf                                    |
| Log4j2             | 2.4.2   | Permet de générer et formatter les logs                      |
| Plugin JaCoCo      | 0.8.5   | Permet de calculer le code coverage (fonctionne avec sonarqube) |

### Tests Unitaires

Afin de réaliser les tests unitaires sur le projet, nous avons utilisé Junit 4.13.1 associé à maven pour les exécuter. Nous utilisons également le plugin JaCoCo afin de calculer le code coverage et le stocker sous un format XML qui sera utilisé par SonarQube.

* Les tests doivent être lancée depuis le container docker, voici la commande à jouer : 

```shell
docker exec api_backend mvn -B -f /home/app/pom.xml test
```

* Afin d'obtenir un rapport du coverage à l'aide de JaCoCo, éxécuter la commande suivante :

```shell
docker exec api_backend mvn -B -f /home/app/pom.xml jacoco:report
```

Puis ouvrez le fichier "target/site/jacoco/index.html" dans votre navigateur.

### Outils de qualité du code

Sonarqube 

* Indiquer la conf pour expliquer comme la changer (pom.xml) => source a inclure ou exclure
* Indiquer les règles importantes/bloquantes par défault et si il y en a, les customs


### Intégration continue

* Redmine : Expliquer comment utiliser jenkins (créer un compte, lancer le build, ...)

* Expliquer la stratégie du build (décrire jenkinsfile)
* Indiquer que c'est automatique (hook) + autres règles s'il y en a


### Déploiement

* à ajouter sur redmine de façon précise (ici on a déjà ce qu'il faut avec l'integration continue)



## Installation et utilisation de l'environnement de développement front-end

* Cloner le repository GitHub suivant : https://github.com/Nicolas-Chambon/mspr_recipe_front
* Pour l'environnement local, utiliser le fichier `docker-compose.yml`

```yml
version: '3'
services:
  client:
    container_name: mspr-recipe-frontend
    build:
      context: .
      dockerfile: dockerfiles/local/Dockerfile
    volumes:
      - ./:/usr/app
      - /usr/app/node_modules
    ports:
      - 4200:4200
```

> Ce fichier **docker-compose** permet de :
>
> * Pour le service `backend``
>   * Construire un conteneur appelé `mspr-recipe-frontend` à partir du **Dockerfile** présent à la racine du projet`
>   * Utiliser la racine du répertoire local comme **volumes** et le lier au source du container
>   * Rediriger le port **4200** du container vers le **4200** de la machine parent



* Pour construire le container et le déployer en local

```bash
docker-compose up --build -d
```

* L'Application est disponible à l'URL suivant : 

```html
http://localhost:4200
```

> **NB :** Les sources local sont liées à celle présente dans le container, du coup pas besoin de build de nouveau à chaque changement dans le code.



### Dépendances

| Dépendance | Version | Commentaire                                                  |
| :--------- | ------: | ------------------------------------------------------------ |
| Angular    |      11 | Framework typescript                                         |
| rxjs       |   6.6.0 | librairie permettant de faciliter la gestion des évènements asynchrones |
| tslib      |   2.0.0 | Il s'agit d'une bibliothèque d'exécution pour TypeScript qui contient toutes les fonctions d'assistance TypeScript. |
| jest       |  26.6.3 | Framework de test javascript                                 |
| prettier   |   2.2.1 | Permet de formaté le code d'un projet selon des règles spécifiques |
| tslint     |   6.1.0 | TSLint est un outil d'analyse statique extensible qui vérifie le code TypeScript pour la lisibilité, la maintenabilité et les erreurs de fonctionnalité |
| husky      |   4.3.8 | Husky est une librairie permettant de faciliter la création et le partage des hooks au sein d’un projet |
| typescript |   4.0.2 | TypeScript est un langage de programmation libre et open source développé par Microsoft |

### Commande utiles

- ng
- start
- start:dev
- build
- build:prod
- test
- test:ci
- test:update
- test:watch
- lint
- format
- sonar
- e2e

### Tests Unitaires



### Outils de qualité du code

Prettier

Tslint

Sonarqube 

* Indiquer la conf pour expliquer comme la changer (pom.xml) => source a inclure ou exclure
* Indiquer les règles importantes/bloquantes par défault et si il y en a, les customs


### Intégration continue

* Redmine : Expliquer comment utiliser jenkins (créer un compte, lancer le build, ...)

* Expliquer la stratégie du build (décrire jenkinsfile)
* Indiquer que c'est automatique (hook) + autres règles s'il y en a


### Déploiement

* à ajouter sur redmine de façon précise (ici on a déjà ce qu'il faut avec l'integration continue)





### hooks avec Husky

> **NB :** Un hook est un script qui s’exécute automatiquement lorsqu’un événement particulier se produit dans un dépôt git.

- #### pre-commit (Ce hook se déclenche en premier avant même de saisir le message du commit)

  - npm run format
    - Formate le code à l'aide de prettier avant de valider un commit
  - npm run lint
    - Lance un linter sur le code afin de vérifier toute erreur potentiel avant de valider le commit

- #### pre-push (Ce hook se déclenche avant l’exécution de la commande git push)

  - npm run test
    - Lance les tests sur le projet avant de push
  - npm run build
    - Tente de build le projet avant de push

- #### Contourner les hooks

  - Si besoin il est possible de contourner l’utilisation des hooks via l’option --no-verify
