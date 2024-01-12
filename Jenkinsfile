pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "mehdilucasammar" // replace this with your docker-id
DOCKER_MOVIE_IMAGE = "movie-service-exam"
DOCKER_CAST_IMAGE = "cast-service-exam"
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
BRANCH_NAME = "${GIT_BRANCH.split("/")[1]}" // See: https://stackoverflow.com/questions/42383273/get-git-branch-name-in-jenkins-pipeline-jenkinsfile
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
                    sh 'docker stop jenkins-movie-service || true'
                    sh 'docker rm jenkins-movie-service || true'
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
                    docker run -d -p 8000:8000 --name jenkins-movie-service $DOCKER_ID/$DOCKER_MOVIE_IMAGE:$DOCKER_TAG
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
                    sh 'docker stop jenkins-cast-service || true'
                    sh 'docker rm jenkins-cast-service || true'
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
                    docker run -d -p 8000:8000 --name jenkins-cast-service $DOCKER_ID/$DOCKER_CAST_IMAGE:$DOCKER_TAG
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
                docker rm -f jenkins
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/$DOCKER_CAST_IMAGE:$DOCKER_TAG
                '''
                }
            }
        }
        stage('Deploiement en dev'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp jenkins_helm_exam/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                cat values.yml
                helm upgrade --install app jenkins_helm_exam --values=values.yml --namespace dev
                '''
                }
            }
        }
        stage('Deploiement en QA'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp jenkins_helm_exam/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app jenkins_helm_exam --values=values.yml --namespace qa
                '''
                }
            }
        }
        stage('Deploiement en staging'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp jenkins_helm_exam/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app jenkins_helm_exam --values=values.yml --namespace staging
                '''
                }
            }
        }
        stage('Deploiement en prod') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    // Check if the branch is master
                    def isMasterBranch = BRANCH_NAME == 'master'
        
                    // If on master, request manual approval
                    if (isMasterBranch) {
                        timeout(time: 15, unit: "MINUTES") {
                            input message: 'Do you want to deploy in production?', ok: 'Yes'
                        }
                        script {
                        sh '''
                        rm -Rf .kube
                        mkdir .kube
                        ls
                        cat $KUBECONFIG > .kube/config
                        cp jenkins_helm_exam/values.yaml values.yml
                        cat values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                        helm upgrade --install app jenkins_helm_exam --values=values.yml --namespace prod
                        '''
                        }
                    } else {
                        echo "Not deploying to production because the build is not on the master branch, branch name:"
                        echo BRANCH_NAME
                    }
                }
            }
        }

    }
}
