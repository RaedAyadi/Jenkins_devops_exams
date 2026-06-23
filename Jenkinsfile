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
            steps {
                script {
                    sh '''
                    docker rm -f movie-service cast-service nginx
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE_MOVIE_SERVICE:$DOCKER_TAG ./movie-service
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE_CAST_SERVICE:$DOCKER_TAG ./cast-service
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE_NGINX:$DOCKER_TAG ./Dockerfile
                    sleep 10
                    '''
                }
            }
        }

    }

}