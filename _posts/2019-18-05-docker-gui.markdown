---
layout: post
title:  "GUI avec Docker (Ubuntu)"
date:   2019-18-05 16:00:00 +0100
categories: Développement Docker
---
![logo docker](/assets/docker-logo.png)

# Introduction

Dans ce post, je vais vous montrer comment lancer une application avec interface graphique depuis un container docker. 
Nous allons utiliser une machine hôte sous ubuntu, et lancer un container avec une image ubuntu également.

Voici le déroulement de l'article :

- Installation de docker

- Configuration de la machine hôte

- Lancement d'une application graphique dans un container

Commençons avec l'installation.

# Installation de docker

L'installation de docker sur ubuntu nécessite de suivre les instructions de la [documentation officielle](https://docs.docker.com/install/linux/docker-ce/ubuntu/).
Le paquet dans les repos ubuntu de base n'est pas forcément à jour, c'est pourquoi il faut passer par le site officiel.
Pour faciliter les choses, je vous mets les commandes à exécuter pour installer docker ci-dessous :

```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

Vous pouvez lancer la commande `sudo docker run hello-world` pour valider que tout fonctionne correctement.
Vous devriez avoir ce résultat :

![docker hello-world](/assets/docker-hello-world.png)

# Configuration

Pour pouvoir lancer des applications en tant que root (dans le container),
il est nécessaire d'exécuter la commande suivante sur votre machine hôte:

```console
user@host:~$ xhost +local:docker
```

Cette commande permet d'autoriser docker à accéder au serveur X (serveur d'affichage sous Unix) de la machine hôte.
Cependant, cela rompt l'isolation entre l'hôte et le container, ce qui peut être dangereux.
En effet, si l'un de vos containers est compromis par un logiciel malveillant
(pas seulement celui contenant les applications en interface graphique),
un pirate pourrait prendre la main sur le serveur X de votre machine hôte.
Ce cas de figure est peu probable, mais il est importante de l'avoir à l'esprit.
D'autres méthodes plus complexes permettent de s'assurer d'une meilleure isolation : voir https://wiki.ros.org/docker/Tutorials/GUI#Using_X_server.

Vous pouvez exécuter la commande ```xhost -local:docker``` à la fin de votre utilisation du container pour réactiver l'isolation.

**Si vous n'allez lancer que des applications en tant qu'utilisateur non privilégié,
il n'est pas nécessaire de faire la commande ```xhost```, et donc pas de problème d'isolation.**

# Lancement d'une application avec interface graphique dans un container

Nous allons voir maintenant la partie intéressante.
Voici la commande pour lancer un container avec la possibilité d'ouvrir des applications graphiques.
Nous utilisons une image ubuntu.

```console
user@host:~$ sudo docker run -it -e DISPLAY --net=host ubuntu
```

Explications :

- `-it` permet d'avoir une session interactive, c'est-à-dire un "beau" shell.

- `-e DISPLAY` permet de passer la variable d'environnement `$DISPLAY` depuis l'environnement hôte vers l'environnement container.

- `--net=host` permet de partager le réseau entre l'hôte et le container.
Cela permet au container d'échanger avec la socket du serveur X.
**De plus, cela forward les ports.
Par exemple, une application qui tourne en localhost sur le port 80 sur le container sera accessible sur l'hôte sur le port 80.
On perd encore une fois l'isolation.**
Si cela est un problème, on peut remplacer `--net=host` par `-v /tmp/.X11-unix:/tmp/.X11-unix`.
Seule la socket du serveur X est ainsi partagée.

# Test

Pour tester que nous avons bien activé les applications graphiques, nous allons installer et lancer xeyes.

```console
root@container:/# apt update && apt install -y x11-apps && xeyes
```

Résultat :

![docker xeyes](/assets/docker-xeyes.png)

# Conclusion

Nous avons pu créer un container docker à partir duquel nous pouvons lancer des applications avec interfaces graphiques.
Les performances ne sont évidemment pas les mêmes que si les applications étaient lancées sur l'hôte, mais cela reste utilisable.
Cela permet de faire des essais sans pour autant installer des logiciels sur notre machine hôte.

N'hésitez pas à me faire part de vos questions ou commentaires concernant cet article, je serai ravi de vous répondre.

# References

https://docs.docker.com/install/linux/docker-ce/ubuntu/

https://forums.docker.com/t/start-a-gui-application-as-root-in-a-ubuntu-container/17069

http://fabiorehm.com/blog/2014/09/11/running-gui-apps-with-docker/

https://wiki.ros.org/docker/Tutorials/GUI
