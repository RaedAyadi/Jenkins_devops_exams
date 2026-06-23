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

        stage ('Docker Build Containers') { // docker build image stage
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

        stage ('Run Docker Containers') { // run containers from the build images
            parallel {
                stage('Run Movie Service') {
                    steps {
                        script {
                            sh '''
                            docker run -d -p 8001:8000 --name movie-service $DOCKER_ID/$DOCKER_IMAGE_MOVIE_SERVICE:$DOCKER_TAG
                            sleep 3
                            '''
                        }
                    }
                }
                stage('Run Cast Service') {
                    steps {
                        script {
                            sh '''
                            docker run -d -p 8002:8000 --name cast-service $DOCKER_ID/$DOCKER_IMAGE_CAST_SERVICE:$DOCKER_TAG
                            sleep 3
                            '''
                        }
                    }
                }
                stage('Run Nginx') {
                    steps {
                        script {    
                            sh '''
                            docker run -d -p 80:80 --name nginx $DOCKER_ID/$DOCKER_IMAGE_NGINX:$DOCKER_TAG
                            sleep 3
                            '''
                        }
                    }
                }
            }

        }

    }

}