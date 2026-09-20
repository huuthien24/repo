pipeline { 
    agent any 

 
    environment { 
        DOCKER_REGISTRY = "docker.io" 
        IMAGE_NAME = "huuthien24/simple-nginx" 
        GIT_CREDENTIAL = "git-cred" 
        DOCKER_CREDENTIAL = "docker-cred" 
    } 

 
    stages { 
        stage('Checkout') { 
            steps { 
                git credentialsId: "${GIT_CREDENTIAL}", url: "https://github.com/huuthien24/repo.git" 
            } 
        } 

 
        stage('Build Docker Image') { 
            steps { 
                script { 
                    IMAGE_TAG = "build-${env.BUILD_NUMBER}" 
                    sh """ 
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ./src 
                    """ 
                } 
            } 
        } 

 
        stage('Push Docker Image') { 
            steps { 
                script { 
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIAL}", 
                            usernameVariable: 'USER', passwordVariable: 'PASS')]) { 
                        sh """ 
                        echo $PASS | docker login -u $USER --password-stdin ${DOCKER_REGISTRY} 
                        docker push ${IMAGE_NAME}:${IMAGE_TAG} 
                        """ 
                    } 
                } 
            } 
        } 

 
        stage('Update Helm values.yaml') { 
            steps { 
                script { 
                    sh """ 
                    sed -i 's/tag:.*/tag: "${IMAGE_TAG}"/' charts/nginx/values.yaml 
                    """ 
                } 
            } 
        } 

 
        stage('Commit & Push Back') { 
            steps { 
                script { 
                    sh """ 
                        git config user.email "jenkins@ci.com" 
                        git config user.name "Jenkins CI" 
                        git add charts/nginx/values.yaml 
                        git commit -m "Update image tag to ${IMAGE_TAG}" 
                        git push origin main 
                    """ 
                } 
            } 
        } 
    } 
} 
