 DATASCIENTEST JENKINS EXAM
#8000 est le port interne utilisé par Docker et que les services écoutent (movie et cast)
Nginx écoute à l'intérieur de son conteneur (ici, 8082)
#8001 et 8002 sont les ports externes 
 port externe est celui par lequel Nginx est exposé à l'hôte (ici, 8081 

# python-microservice-fastapi
Learn to build your own microservice using Python and FastAPI

## How to run??
 - Make sure you have installed `docker` and `docker-compose`
docker --version
docker-compose --version
sudo apt update
sudo apt install docker-compose

#nettoyer conteneurs et images
docker-compose down
docker rm -f $(docker ps -aq)
docker image prune -a
docker system prune -a


 - Run `docker-compose up -d`
 - Head over to http://localhost:8001/api/v1/movies/docs for movie service docs 
   and http://localhost:8002/api/v1/casts/docs for cast service docs
#Pour créer les quatre environnements dev, QA, staging, et prod sur un cluster Kubernetes, vous utiliserez les Namespaces, qui sont des espaces isolés dans Kubernetes permettant de gérer des environnements indépendants au sein du même cluster. Voici comment procéder :
#1. Créer les Namespaces Kubernetes pour chaque Environnement
kubectl create namespace dev
kubectl create namespace qa
kubectl create namespace staging
kubectl create namespace prod

#Vous pouvez vérifier que les namespaces sont bien créés en exécutant :
kubectl get namespaces

se connecter docker login

construire le build (se placer dans movie-service pour la seconde image)
docker docker build -t julh62/jenkins-devops-exams-movie-service:latest .
docker push julh62/jenkins-devops-exams-movie-service:latest



pousser l'image sur dockerhub
ou bien pull l'image si elle est deja sur dockerhub 

# meme exercice en utulisant kubernetes
#creer les fichiers .yaml de deployment de de service
# crée un ConfigMap nommé nginx-config, dans lequel nginx-config.conf est une clé, et le contenu du fichier est sa valeur.
kubectl create configmap nginx-config --from-file=nginx-config.conf
kubectl delete configmap --all -n default si beoin avant de créer

# apply tous les manifestes
Étape 1: Recréer les ConfigMaps

Comme vous avez supprimé les ressources, assurez-vous d'avoir les ConfigMaps nécessaires pour vos services,
 comme cast-service-config, movie-service-config.yaml, 
 et cast-db-config.

#Recréez les ConfigMaps si ce n'est pas déjà fait. Par exemple :

kubectl apply -f cast-service-config.yaml
kubectl apply -f cast-db-config.yaml
kubectl apply -f movie-service-config.yaml

#Étape 2: Recréer les PVCs
kubectl get pvc -n default
kubectl delete pvc --all -n default
kubectl apply -f cast-db-volumes.yaml
kubectl apply -f movie-db-volumes.yaml

#Étape 3: Recréer les Déploiements (Deployments)
kubectl delete deployments --all -n default  si besoin
Une fois les PVCs et ConfigMaps créés, vous pouvez appliquer les fichiers de déploiement pour chaque service. Appliquez les fichiers YAML des déploiements comme suit :

kubectl apply -f cast-db-deployment.yaml
kubectl apply -f movie-db-deployment.yaml
kubectl apply -f movie-service-deployment.yaml
kubectl apply -f cast-service-deployment.yaml
kubectl apply -f nginx-deployment.yaml

#Étape 4: Recréer les Services
kubectl delete svc --all -n default
Une fois les déploiements créés, vous pouvez appliquer les fichiers YAML des services associés pour chaque application. Par exemple :

kubectl apply -f cast-service-service.yaml
kubectl apply -f movie-service-service.yaml
kubectl apply -f nginx-service.yaml
kubectl apply -f cast-db-service.yaml

#Étape 5: Vérification

Une fois toutes les ressources appliquées, vérifiez que tout est en place et fonctionne correctement en exécutant :

kubectl get all
# 
#se connecter au conteneur postgres (cast-db)
kubectl exec -it cast-db-dbbd5874c-6df89 -- bash
find / -name "pg_hba.conf"
vi /var/lib/postgresql/data/pg_hba.conf
ajouter une ligne dans pg_hba.conf dans ipv4
Résumé des touches importantes dans vi :

    i : Passer en mode insertion (avant le curseur).
    a : Passer en mode insertion (après le curseur).
    Esc : Quitter le mode d'insertion et revenir en mode commande.
    :wq : Sauvegarder et quitter.
    :q! : Quitter sans sauvegarder.
host    all             all             10.43.137.0/24           trust
#redemarrer le conteneur 
kubectl exec -it cast-db-dbyuhyh -- pg_ctl restart -D /var/lib/postgresql/data
