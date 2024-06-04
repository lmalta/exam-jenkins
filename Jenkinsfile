pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "lmalta3" // replace this with your docker-id
DOCKER_IMAGE_CAST = "cast-service"
DOCKER_IMAGE_MOVIE = "movie-service"
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
}
agent any // Jenkins will be able to select all available agents
stages {
        stage(' Docker Build'){ // docker build image stage
            steps {
                script {
                sh '''
                 /Users/louismalta/.rd/bin/docker rm -f jenkins
                 /Users/louismalta/.rd/bin/docker build -t $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG cast-service/
                 /Users/louismalta/.rd/bin/docker build -t $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG movie-service/
                sleep 6
                '''
                }
            }
        }
//        stage('Docker run'){ // run container from our builded image
//                steps {
//                    script {
//                    sh '''
//                    docker run -d -p 80:80 --name jenkins $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG
//                    docker run -d -p 80:80 --name jenkins $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG
//                    sleep 10
//                    '''
//                    }
//                }
//            }

        stage('Test Acceptance'){ // we launch the curl command to validate that the container responds to the request
            steps {
                    script {
                    sh '''
                    curl localhost
                    '''
                    }
            }

        }
        stage('Docker Push'){ //we pass the built image to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }

            steps {

                script {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG
                docker push $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG
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
                helm upgrade --install app prj/ --values=prj/values.yaml --namespace dev  --set movie.image.repository=$DOCKER_IMAGE_MOVIE --set movie.image.tag=$DOCKER_TAG --set cast.image.repository=$DOCKER_IMAGE_CAST --set cast.image.tag=$DOCKER_TAG
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
                helm upgrade --install app prj/ --values=prj/values.yaml --namespace staging  --set movie.image.repository=$DOCKER_IMAGE_MOVIE --set movie.image.tag=$DOCKER_TAG --set cast.image.repository=$DOCKER_IMAGE_CAST --set cast.image.tag=$DOCKER_TAG
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
                helm upgrade --install app prj/ --values=prj/values.yaml --namespace qa  --set movie.image.repository=$DOCKER_IMAGE_MOVIE --set movie.image.tag=$DOCKER_TAG --set cast.image.repository=$DOCKER_IMAGE_CAST --set cast.image.tag=$DOCKER_TAG
                '''
                }
            }

        }
  stage('Deploiement en prod'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
            // Create an Approval Button with a timeout of 15minutes.
            // this require a manuel validation in order to deploy on production environment
                    timeout(time: 15, unit: "MINUTES") {
                        input message: 'Do you want to deploy in production ?', ok: 'Yes'
                    }

                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                helm upgrade --install app prj/ --values=prj/values.yaml --namespace prod --set movie.image.repository=$DOCKER_IMAGE_MOVIE --set movie.image.tag=$DOCKER_TAG --set cast.image.repository=$DOCKER_IMAGE_CAST --set cast.image.tag=$DOCKER_TAG
                '''
                } 
            }

        }

}
}
