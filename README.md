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

# si besoin , si les bases de données ne sont pas crées
Installer psql dans le conteneur

    Accédez au conteneur où vous voulez utiliser psql (dans ce cas movie-service ou un autre).

docker exec -it jenkins_devops_exams_movie-service_1 bash

Installez le client PostgreSQL :

    Si le conteneur est basé sur Debian/Ubuntu :
si besoin installer psql sur les conteneurs movie-service et cast-service
apt-get update && apt-get install -y postgresql-client
psql -h cast-db -U postgres -W
-- Créer la base de données (si elle n'existe pas déjà)
CREATE DATABASE cast_db_dev;

-- Créer l'utilisateur (si nécessaire)
CREATE USER cast_db_username WITH PASSWORD 'cast_db_password';

-- Donner tous les privilèges sur la base de données à l'utilisateur
GRANT ALL PRIVILEGES ON DATABASE cast_db_dev TO cast_db_username;

-- Vous pouvez aussi donner à cet utilisateur les privilèges nécessaires pour les tables qui seront créées plus tard :
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO cast_db_username;


pousser l'image sur dockerhub
ou bien pull l'image si elle est deja sur dockerhub 

# meme exercice en utulisant kubernetes
#creer les fichiers .yaml de deployment de de service
# crée un ConfigMap nommé nginx-config, dans lequel nginx-config.conf est une clé, et le contenu du fichier est sa valeur.
kubectl get configmap -n default
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
kubectl apply -f movie-db-config.yaml
kubectl apply -f nginx-service.yaml

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
kubectl apply -f movie-db-service.yaml

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


## installer Ingress pour aider à la configuration des routes
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml

kubectl get pods -n ingress-nginx
kubectl apply -f movie-service-ingress.yaml
kubectl apply -f cast-service-ingress.yaml





### Déploiement avec Helm

1. Installer Helm :
   ```bash
   curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
   chmod 700 get_helm.sh
   ./get_helm.sh

    Créer les fichiers de configuration (facultatif) :
        Modifiez fastapiapp/values.yaml pour personnaliser les paramètres du déploiement.

    Déployer sur Kubernetes :

helm install fastapiapp ./fastapiapp

Mettre à jour le déploiement si nécessaire :

helm upgrade fastapiapp ./fastapiapp

Supprimer le déploiement :

    helm uninstall fastapiapp


---
Option 1 : Conservez-le en dehors du chart Helm

Laissez kubeconfig.yaml à la racine de votre projet, où il est actuellement. Vous pouvez configurer Helm pour utiliser ce fichier si nécessaire, par exemple :

Ensuite, toutes vos commandes helm et kubectl utiliseront ce fichier.
Option 2 : Configuration par défaut

Si votre cluster est déjà configuré avec kubectl (par exemple, via le fichier par défaut ~/.kube/config), vous n'avez pas besoin d'un fichier kubeconfig.yaml séparé. Dans ce cas, vous pouvez simplement supprimer ce fichier si vous êtes sûr de ne plus en avoir besoin.
Résumé

    Ne déplacez pas kubeconfig.yaml dans le chart Helm.
    Utilisez-le comme configuration externe pour accéder au cluster.
    Configurez votre environnement avec export KUBECONFIG=/path/to/kubeconfig.yaml si nécessaire.


Vérification et déploiement

Déploiement

#    Testez le fichier avec :


helm lint .

Cela vérifiera la syntaxe, les placeholders, et les valeurs utilisées dans values.yaml.

Ce que helm lint vérifie :

    La syntaxe des fichiers YAML dans les templates.
    La validité des valeurs dans values.yaml.
    La structure du chart Helm.
    La cohérence des références entre les fichiers.

Résultat :

    Si tout est correctement configuré, vous devriez obtenir un message de succès avec des recommandations (s'il y en a).
    Si des erreurs sont détectées, Helm vous indiquera où elles se trouvent pour que vous puissiez les corriger.

root@ubuntu:/srv/jenkins-exam/fastapiapp# helm lint .
==> Linting .
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed
root@ubuntu:/srv/jenkins-exam/fastapiapp# 

ChatGPT a dit :
ChatGPT

La sortie de la commande helm lint . indique que votre chart est valide :

1 chart(s) linted, 0 chart(s) failed

Cela signifie que tous les fichiers dans votre chart Helm sont correctement configurés et ne contiennent pas d'erreurs de syntaxe ou de configuration. Le message [INFO] Chart.yaml: icon is recommended est simplement une suggestion que vous pouvez ajouter une icône pour votre chart, mais cela n'affecte pas le bon fonctionnement du chart.
Prochaines étapes :

