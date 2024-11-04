# DATASCIENTEST JENKINS EXAM
# python-microservice-fastapi
Learn to build your own microservice using Python and FastAPI

## How to run??
 - Make sure you have installed `docker` and `docker-compose`
docker --version
docker-compose --version
sudo apt update
sudo apt install docker-compose


 - Run `docker-compose up -d`
 - Head over to http://localhost:8080/api/v1/movies/docs for movie service docs 
   and http://localhost:8080/api/v1/casts/docs for cast service docs
#Pour créer les quatre environnements dev, QA, staging, et prod sur un cluster Kubernetes, vous utiliserez les Namespaces, qui sont des espaces isolés dans Kubernetes permettant de gérer des environnements indépendants au sein du même cluster. Voici comment procéder :
#1. Créer les Namespaces Kubernetes pour chaque Environnement
kubectl create namespace dev
kubectl create namespace qa
kubectl create namespace staging
kubectl create namespace prod

Vous pouvez vérifier que les namespaces sont bien créés en exécutant :

bash

kubectl get namespaces