pipeline {

    environment {
        // Declaration of environment variables
        DOCKER_ID = "raedayadi"           // replace this with your docker-id
        DOCKER_IMAGE_MOVIE_SERVICE = "movie-service"
        DOCKER_IMAGE_CAST_SERVICE = "cast-service"
        DOCKER_IMAGE_NGINX = "nginx"
        DOCKER_TAG = "v.${BUILD_ID}.0"  // we will tag our images with the current build in order to increment the value by 1 with each new build
    }

    agent any 

    stages {

        stage ('Docker Build') { // docker build image stage
            parallel {
                stage('Build Movie Service') {
                    steps {
                        script {
                            sh '''
                            docker rm -f movie-service
                            docker build -t $DOCKER_ID/$DOCKER_IMAGE_MOVIE_SERVICE:$DOCKER_TAG ./movie-service
                            sleep 3
                            '''
                        }
                    }
                }
                stage('Build Cast Service') {
                    steps {
                        script {
                            sh '''
                            docker rm -f cast-service
                            docker build -t $DOCKER_ID/$DOCKER_IMAGE_CAST_SERVICE:$DOCKER_TAG ./cast-service
                            sleep 3
                            '''
                        }
                    }
                }
                stage('Build Nginx') {
                    steps {
                        script {    
                            sh '''
                            docker rm -f nginx
                            docker build -t $DOCKER_ID/$DOCKER_IMAGE_NGINX:$DOCKER_TAG -f ./Dockerfile .
                            sleep 3
                            '''
                        }
                    }
                }
            }

        }

    }

}