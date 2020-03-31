# Pacman + NGINX + PHP + MongoDB Application

This is an example Kubernetes application that hosts an HTML5 Pacman game with NGINX as the web server and PHP backend to read
and write data to a MongoDB database.

## Pacman Game Architecture Overview

### Pacman

The Pacman game is a slightly modified version of the open source Pacman game written in HTML5 with Javascript. You can get the
modified [Pacman game source code here](https://github.com/font/pacman-canvas).

### NGINX + PHP FPM

[NGINX](https://www.nginx.com/) is used as the web server to host the Pacman game application. It is configured with
[PHP FPM](https://php-fpm.org/) support for the backend PHP API.

### PHP

PHP FPM is used for the PHP API to receive read and write requests from clients and perform database operations. The
[PHP MongoDB driver extension](http://php.net/manual/en/set.mongodb.php) contains the minimal API for core driver functionality. In addition, the
[PHP library for MongoDB](http://php.net/manual/en/mongodb.tutorial.library.php) provides the higher level APIs. Both are needed.

### MongoDB

[MongoDB](https://www.mongodb.com/) is used as the backend database to store the Pacman game's high score user data.

## Prerequisites

### Clone This Repository

Clone this repo which contains the Kubernetes configs and Dockerfile resources.

```
git clone https://github.com/mirantis-field/pacman.git
cd pacman/pacman-nginx-app
```

## Create Application Container Image

The [Dockerfile](docker/Dockerfile) performs the following steps:

1. It is based on Ubuntu 16.04 Xenial and installs NGINX, PHP FPM, PHP MongoDB, and Composer (for the PHP MongoDB lib).
2. It then creates a basic NGINX configuration that includes enabling PHP FastCGI.
3. Clones the Pacman game into the configured root directory of the NGINX web server.
4. Replaces the host 'localhost' with 'mongo' in the PHP backend API to match the host DNS given to the MongoDB Kubernetes service.
5. Exposes port 80 for the web server.
6. Starts `nginx`, `php7.0-fpm`, and runs a forever command to keep the container running.

To build the image run:

```text
export DTR_FQDN=dtr.demo.mirantis.com
export DTR_ORGANIZATION=launch-party

cd docker
docker build -t ${DTR_FQDN}/${DTR_ORGANIZATION}/pacman-nginx-app:latest .
cd ..
```

You can test the image by running:

```text
docker run -p 8000:80 ${DTR_FQDN}/${DTR_ORGANIZATION}/pacman-nginx-app:latest
```

And going to `http://localhost:8000/` to see if you get the Pacman game.

## Container Registry

### Push image to Docker Trusted Registry

You'll want to tag your previously created Docker image to use the Google Cloud Container Registry URL and then push it:

```text
docker login ${DTR_FQDN}
docker push ${DTR_FQDN}/${DTR_ORGANIZATION}/pacman-nginx-app
```

Once you've pushed your image, you'll need to update the Kubernetes resources to point to your image before you continue
with the rest of the guides.

## Kubernetes

```text
sed -i 's/dtr.demo.mirantis.com/${DTR_FQDN}/' controllers/web-controller.yaml replicasets/pacman-replicaset.yaml
sed -i 's/launch-party/${DTR_ORGANIZATION}/' controllers/web-controller.yaml replicasets/pacman-replicaset.yaml
```

### Use-Cases

You'll need to create 1 or at least 3 Kubernetes cluster(s) depending on whether you want to try out the Pac-Man app on 1 cluster,
or try it out on a federated cluster. Below are links to the two choices that will guide you through it:

- [Pac-Man NGINX App Single Cluster](docs/pacman-nginx-app-single-cluster.md)
- [Pac-Man NGINX App Federated Cluster](docs/pacman-nginx-app-federated-cluster.md)
