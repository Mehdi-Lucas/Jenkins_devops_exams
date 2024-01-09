pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "mehdilucasammar" // replace this with your docker-id
DOCKER_MOVIE_IMAGE = "movie-service-exam"
DOCKER_CAST_IMAGE = "cast-service-exam"
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
}
agent any // Jenkins will be able to select all available agents
stages
    {
        stage('Docker Build MOVIE SERVICE')
        { // docker build movie service image stage
            steps
            {
                script 
                {
                    sh '''
                    cd movie-service
                    docker build -t $DOCKER_ID/$DOCKER_MOVIE_IMAGE:$DOCKER_TAG -f Dockerfile .
                    '''
                }
            }
        }
        stage('Docker run MOVIE SERVICE'){ // run container from our builded image
                steps {
                    script {
                    sh '''
                    docker run -d -p 8000:8000 --name jenkins $DOCKER_ID/$DOCKER_MOVIE_IMAGE:$DOCKER_TAG
                    sleep 10
                    '''
                    }
                }
            }

        stage('Test Acceptance MOVIE SERVICE'){ // we launch the curl command to validate that the container responds to the request
            steps {
                    script {
                    sh '''
                    curl localhost
                    '''
                    }
            }

        }
        stage('Docker Push MOVIE SERVICE'){ //we pass the built image to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }

            steps {

                script {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/$DOCKER_MOVIE_IMAGE:$DOCKER_TAG
                '''
                }
            }

        }
        stage('Docker Build CAST SERVICE')
        { // docker build cast service image stage
            steps  
            {
                script 
                {
                    sh '''
                    cd cast-service
                    docker build -t $DOCKER_ID/$DOCKER_CAST_IMAGE:$DOCKER_TAG -f Dockerfile .
                    '''
                }
            }
        }
        stage('Docker run CAST SERVICE'){ // run container from our builded image
                steps {
                    script {
                    sh '''
                    docker run -d -p 8000:8000 --name jenkins $DOCKER_ID/$DOCKER_CAST_IMAGE:$DOCKER_TAG
                    sleep 10
                    '''
                    }
                }
            }

        stage('Test Acceptance CAST SERVICE'){ // we launch the curl command to validate that the container responds to the request
            steps {
                    script {
                    sh '''
                    curl localhost
                    '''
                    }
            }

        }
        stage('Docker Push CAST SERVICE'){ //we pass the built image to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }

            steps {

                script {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/$DOCKER_CAST_IMAGE:$DOCKER_TAG
                '''
                }
            }
        }
    }
}
