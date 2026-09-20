pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'huuthien24/simple-nginx' // Thay bằng tên repo Docker Hub cua bạn
        BUILD_TAG = "build-${BUILD_NUMBER}"
        GIT_REPO_URL = 'https://github.com/huuthien/repo.git' // Thay URL Git cua bạn
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    // Login và Build/Push Image lên Docker Hub
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credentials') {
                        def customImage = docker.build("${DOCKER_IMAGE}:${BUILD_TAG}")
                        customImage.push()
                        customImage.push('latest')
                    }
                }
            }
        }

        stage('Update Helm Values & Push to Git') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github-credentials', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                        // Cập nhật tag mới vào values.yaml su dụng sed
                        sh """
                            sed -i 's/tag: .*/tag: "${BUILD_TAG}"/' charts/nginx/values.yaml
                            
                            git config user.email "jenkins@ci.com"
                            git config user.name "Jenkins CI"
                            
                            git add charts/nginx/values.yaml
                            git commit -m "ci: update image tag to ${BUILD_TAG} [skip ci]"
                            
                            # Push thay đoi ve lại GitHub
                            git push https://${GIT_USER}:${GIT_PASS}@github.com/huuthien24/repo.git HEAD:main
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "CI Pipeline thành công! Tag mới: ${BUILD_TAG}"
        }
        failure {
            echo "CI Pipeline that bại!"
        }
    }
}
