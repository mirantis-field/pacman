# Pac-Man + NodeJS + MongoDB Microservices Application

This is an example Kubernetes application that hosts an HTML5 Pac-Man game with Node.js as the web server and backend to read
and write data to a MongoDB database.

## Pac-Man Game Architecture Overview

### Pac-Man

The Pac-Man game is a modified version of the open source Pac-Man game written in HTML5 with Javascript. You can get the
modified [Pac-Man game source code here](https://github.com/mirantis/pacman-nodejs.git).

### Node.js

[Node.js](https://nodejs.org/) is used as the server side component to host the Pac-Man game application. It uses a few packages such as the
[Express](https://expressjs.com/) web application framework as well as the [MongoDB](https://mongodb.github.io/node-mongodb-native/) driver
for the backend Node.js API.

### MongoDB

[MongoDB](https://www.mongodb.com/) is used as the backend database to store the Pac-Man game's high score user data.

## Prerequisites

### Clone This Repository

Clone this repo which contains the Kubernetes configs and Dockerfile resources.

```bash
git clone https://github.com/mirantis-field/pacman.git
cd pacman/pacman-nodejs-app
```

## Create Application Container Image

The [Dockerfile](docker/Dockerfile) performs the following steps:

1. It is based on Node.js LTS Version 6 (Boron).
1. It then clones the Pac-Man game into the configured application directory.
1. Exposes port 8080 for the web server.
1. Starts the Node.js application using `npm start`.

To build the image run:

```bash
export DTR_FQDN=dtr.demo.mirantis.com
export DTR_ORGANIZATION=launch-party

cd docker
docker build -t ${DTR_FQDN}/${DTR_ORGANIZATION}/pacman-nodejs:latest .
cd ..
```

You can test the image by running:

```bash
docker run -it --rm -p 8000:8080 ${DTR_FQDN}/${DTR_ORGANIZATION}/pacman-nodejs:latest
```

And going to `http://localhost:8000/` to see if you get the Pac-Man game.

## Container Registry

### Push the Image to Docker Trusted Registry

You'll want to tag your previously created Docker image to use the URL and then push it:

```bash
docker login ${DTR_FQDN}
docker push ${DTR_FQDN}/${DTR_ORGANIZATION}/pacman-nodejs
```

Once you've pushed your image, you'll need to update the Kubernetes resources
to point to your image before you continue with the rest of the guides.

## Kubernetes

```bash
sed -i "s/dtr.demo.mirantis.com/${DTR_FQDN}/" deployments/pacman-deployment*.yaml
sed -i "s/launch-party/${DTR_ORGANIZATION}/" deployments/pacman-deployment*.yaml
```

### Use-Cases

You'll need to create 1 to at least 3 Kubernetes cluster(s) depending on whether you want to try out the Pac-Man app on 1 cluster,
or try it out on multiple clusters. Below are links that will guide you through it.

#### Single Kubernetes Cluster

- [Pac-Man Node.js App Single Cluster](docs/pacman-nodejs-app-single-cluster.md)

#### Federated-v2 Kubernetes Cluster Use-Cases

Follow the instructions in the below links to test out different federation use-cases.

- [Pac-Man application deployed on GKE k8s federated cluster](docs/pacman-nodejs-app-federated-gke.md)
- [Pac-Man application deployed on multiple public cloud providers in a federation: GKE, AWS, and Azure](docs/pacman-nodejs-app-federated-multicloud.md)
- [Pac-Man application portability: deploy on AWS, GKE, and Azure, then move
  away from AWS](docs/pacman-nodejs-app-federated-aws-gke-az-portability.md)
- [Pac-Man application portability: deploy on GKE and Azure, then swap Azure
  with AWS](docs/pacman-nodejs-app-federated-gke-az-aws-portability.md)

#### Kubernetes Cluster Use-Cases (Without Federation)

- [Pac-Man application migration using GKE](docs/pacman-nodejs-app-gke-migration.md)
- [Pac-Man application migration from AWS to GKE](docs/pacman-nodejs-app-aws-gke-migration.md)
