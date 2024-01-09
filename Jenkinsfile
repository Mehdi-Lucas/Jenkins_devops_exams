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
        stage(' Docker Build MOVIE SERVICE')
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

        stage(' Docker Build CAST SERVICE')
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
    }
}
