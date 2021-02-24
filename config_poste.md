# Installation du poste de travail

## Prérequis

* Ubuntu 20;04 installé sur la machine
* Git
* Docker et Docker Compose

## Paquets de base à installer

Afin d'avoir des outils de base commun, nous allons installer par défaut les paquets ubuntu suivants : `git` `vim` `meld` `url` `wget` `ssh`

* Avant d'installer les paquets, faire une mise à jour et redémarrer le PC : 

```shell
sudo apt update && sudo apt upgrade && reboot
```

* Ensuite, voici la commande pour installer les paquets nécessaires : 

```shell
sudo apt install git vim meld curl wget ssh
```


## Outils JetBrains

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

## Installation docker et docker-composer

Docker nous permettra de conteneuriser nos applications et outils de façons simple et efficace, nous utiliserons également docker-compose pour dockeriser nos applications frontend et backend.

> **Installation :**

> * **Installation de docker et docker-compose**

> ```shell
> sudo apt install docker-compose
> ```

> * **Ajouter son user au groupe docker**

> ```shell
> sudo usermod -aG docker $USER
> ```

> * **Redémarrer la session**

> ```shell 
> sudo /sbin/reboot
> ```

> * **Lancer le service Docker** 

> ```shell
> sudo service docker start
> ```
