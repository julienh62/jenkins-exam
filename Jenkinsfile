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
                  timeout(time: 13, unit: 'MINUTES') { 
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
                   timeout(time: 13, unit: 'MINUTES') { 
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

    }
}
