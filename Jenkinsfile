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

        stage('Run Docker Containers') {
            steps {
                sh '''
                docker network create app-network || true

                docker rm -f movie_db cast_db movie_service cast_service nginx || true

                docker run -d --network app-network --name movie_db \
                    -e POSTGRES_USER=movie_db_username \
                    -e POSTGRES_PASSWORD=movie_db_password \
                    -e POSTGRES_DB=movie_db_dev \
                    postgres:12.1-alpine

                docker run -d --network app-network --name cast_db \
                -e POSTGRES_USER=cast_db_username \
                -e POSTGRES_PASSWORD=cast_db_password \
                -e POSTGRES_DB=cast_db_dev \
                postgres:12.1-alpine

                sleep 10

                docker run -d --network app-network --name movie_service -p 8001:8000 \
                -e DATABASE_URI=postgresql://movie_db_username:movie_db_password@movie_db/movie_db_dev \
                -e CAST_SERVICE_HOST_URL=http://cast_service:8000/api/v1/casts/ \
                $DOCKER_ID/$DOCKER_IMAGE_MOVIE_SERVICE:$DOCKER_TAG
                
                sleep 10

                docker run -d --network app-network --name cast_service -p 8002:8000 \
                -e DATABASE_URI=postgresql://cast_db_username:cast_db_password@cast_db/cast_db_dev \
                $DOCKER_ID/$DOCKER_IMAGE_CAST_SERVICE:$DOCKER_TAG
                
                sleep 10

                docker run -d --network app-network --name nginx -p 80:8080 \
                $DOCKER_ID/$DOCKER_IMAGE_NGINX:$DOCKER_TAG
                '''
            }
        }

        stage('Test Acceptance') { // we launch the curl command to validate that the container responds to the request
            parallel {
                stage('Test Movie Service') {
                    steps {
                        script {
                            sh '''
                            curl http://localhost:8001/api/v1/movies
                            '''
                        }
                    }
                }
                stage('Test Cast Service') {
                    steps {
                        script {
                            sh '''
                            curl http://localhost:8002/api/v1/casts
                            '''
                        }
                    }
                }
                stage('Test Nginx') {
                    steps {
                        script {    
                            sh '''
                            curl localhost:80
                            '''
                        }
                    }
                }
            }
        }

        stage('Push Docker Images') { // push the built image to my docker hub account
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // retrieve docker password from secret text called docker_hub_pass saved on jenkins
            }
            steps {
                script {
                    sh '''
                    docker login -u $DOCKER_ID -p $DOCKER_PASS

                    docker push $DOCKER_ID/$DOCKER_IMAGE_MOVIE_SERVICE:$DOCKER_TAG
                    docker push $DOCKER_ID/$DOCKER_IMAGE_CAST_SERVICE:$DOCKER_TAG
                    docker push $DOCKER_ID/$DOCKER_IMAGE_NGINX:$DOCKER_TAG
                    '''
                }
            }
        }

        stage('Deployment in dev') {
            environment {
                KUBECONFIG = credentials("config") // retrieve kubeconfig from secret file called config saved on jenkins
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp charts/values.yaml values.yml
                    cat values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install app charts --values=values.yml --namespace dev
                    '''
                }
            }
        }

        stage('Deployment in qa') {
            environment {
                KUBECONFIG = credentials("config") // retrieve kubeconfig from secret file called config saved on jenkins
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp charts/values.yaml values.yml
                    cat values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install app charts --values=values.yml --namespace qa
                    '''
                }
            }
        }

        stage('Deployment in staging'){
            environment {
                KUBECONFIG = credentials("config") // retrieve kubeconfig from secret file called config saved on jenkins
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp charts/values.yaml values.yml
                    cat values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install app charts --values=values.yml --namespace staging
                    '''
                }
            }
        }

        stage('Deployment in prod'){
            environment {
                KUBECONFIG = credentials("config") // retrieve kubeconfig from secret file called config saved on jenkins
            }
            when {
                branch 'master' // deploy to prod only if the branch is master
            }

            timeout(time: 15, unit: "MINUTES") {
                input message: 'Do you want to deploy in production ?', ok: 'Yes'
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp charts/values.yaml values.yml
                    cat values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install app charts --values=values.yml --namespace prod
                    '''
                }
            }
        }

    }

}