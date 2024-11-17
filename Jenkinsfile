pipeline {
    environment {
        DOCKER_ID = "julh62"  // Votre identifiant DockerHub
        CAST_SERVICE_IMAGE = "jenkins_devops_exams_cast_service"  // Nom de l'image pour cast-service
        MOVIE_SERVICE_IMAGE = "jenkins_devops_exams_movie_service"  // Nom de l'image pour movie-service
        DOCKER_TAG = "v.${BUILD_ID}.0"  // Tag de l'image basé sur l'ID du build
    }
    agent any  // Utilisation d'un agent Jenkins disponible
    stages {
        
        // Étape de construction de l'image pour cast-service
        stage('Docker Build Cast Service') {
            steps {
                script {
                  timeout(time: 1, unit: 'MINUTES') { 
                    sh '''
                    docker rm -f cast-service-container || true
                    docker build -t $DOCKER_ID/$CAST_SERVICE_IMAGE:$DOCKER_TAG -f cast-service/Dockerfile cast-service
                    '''
                }
              }
            }
        }
        
        // Étape de construction de l'image pour movie-service
        stage('Docker Build Movie Service') {
            steps {
                script {
                   timeout(time: 1, unit: 'MINUTES') { 
                    sh '''
                    docker rm -f movie-service-container || true
                    docker build -t $DOCKER_ID/$MOVIE_SERVICE_IMAGE:$DOCKER_TAG -f movie-service/Dockerfile movie-service
                    '''
                }
            }
          }
        }

        // Étape de push des images vers DockerHub
        stage('Docker Push') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")  // Récupération du mot de passe DockerHub depuis les credentials Jenkins
            }
            steps {
                script {
                    sh '''
                    docker login -u $DOCKER_ID -p $DOCKER_PASS
                    docker push $DOCKER_ID/$CAST_SERVICE_IMAGE:$DOCKER_TAG
                    docker push $DOCKER_ID/$MOVIE_SERVICE_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }

        // Étape de test pour valider que les services sont opérationnels
        stage('Test Docker Images') {
            steps {
                script {
                    sh '''
                    # Test du service Cast
                    docker run -d --name cast-service-test -p 8081:8081 $DOCKER_ID/$CAST_SERVICE_IMAGE:$DOCKER_TAG
                    # Attendre que le service soit prêt
                    until $(curl --output /dev/null --silent --head --fail http://localhost:8081); do
                        echo "Waiting for Cast service to be ready..."
                        sleep 5
                    done
                    docker rm -f cast-service-test

                    # Test du service Movie
                    docker run -d --name movie-service-test -p 8080:8080 $DOCKER_ID/$MOVIE_SERVICE_IMAGE:$DOCKER_TAG
                    # Attendre que le service soit prêt
                    until $(curl --output /dev/null --silent --head --fail http://localhost:8080); do
                        echo "Waiting for Movie service to be ready..."
                        sleep 5
                    done
                    docker rm -f movie-service-test
                    '''
                }
            }
        }

        // Déploiement en développement (optionnel)
        stage('Deployment to Dev') {
            environment {
                KUBECONFIG = credentials("kubeconfig_dev")  // Configuration Kubernetes pour l'environnement Dev
            }
            steps {
                script {
                     timeout(time: 1, unit: 'MINUTES') { 
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    helm upgrade --install movie-service movie-service/helm --set image.tag=$DOCKER_TAG --namespace dev
                    helm upgrade --install cast-service cast-service/helm --set image.tag=$DOCKER_TAG --namespace dev
                    '''
                }
              }
            }
        }

        // Déploiement sur le namespace QA
        stage('Deployment to QA') {
            environment {
                KUBECONFIG = credentials("kubeconfig_qa")  // Configuration Kubernetes pour l'environnement QA
            }
            steps {
                script {
                  timeout(time: 1, unit: 'MINUTES') { 
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    helm upgrade --install movie-service movie-service/helm --set image.tag=$DOCKER_TAG --namespace qa
                    helm upgrade --install cast-service cast-service/helm --set image.tag=$DOCKER_TAG --namespace qa
                    '''
                }
            }
          }
        }

        // Optionnel : Déploiement sur d'autres environnements comme staging ou prod
        stage('Deploy to Staging') {
            environment {
                KUBECONFIG = credentials("kubeconfig_staging")
            }
            steps {
                script {
                    timeout(time: 1, unit: 'MINUTES') { 
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    helm upgrade --install movie-service movie-service/helm --set image.tag=$DOCKER_TAG --namespace staging
                    helm upgrade --install cast-service cast-service/helm --set image.tag=$DOCKER_TAG --namespace staging
                    '''
                }
            }
          }
        }

        // Optionnel : Déploiement en production avec validation manuelle
        stage('Deploy to Prod') {
            environment {
                KUBECONFIG = credentials("kubeconfig_prod")
            }
            steps {
                timeout(time: 1, unit: "MINUTES") {
                    input message: 'Do you want to deploy to production?', ok: 'Yes'
                }
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    helm upgrade --install movie-service movie-service/helm --set image.tag=$DOCKER_TAG --namespace prod
                    helm upgrade --install cast-service cast-service/helm --set image.tag=$DOCKER_TAG --namespace prod
                    '''
                }
            }
        }
    }
}
