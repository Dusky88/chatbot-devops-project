pipeline {
    agent none
    
    environment {
        IMAGE_NAME = "dusky88/chatbot"
        IMAGE_TAG  = "${BUILD_NUMBER}"
        APP_DIR    = "Chatbot-UI"
    }
    
    stages {
        stage('Build & Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '-v /var/run/docker.sock:/var/run/docker.sock -v /usr/bin/docker:/usr/bin/docker'
                }
            }
            stages {
                stage('Checkout') {
                    steps {
                        git branch: 'staging',
                            credentialsId: 'github-creds',
                            url: 'https://github.com/Dusky88/chatbot-devops-project.git'
                    }
                }
                stage('Install Dependencies') {
                    steps {
                        dir("${APP_DIR}") {
                            sh 'npm ci --cache /tmp/npm-cache'
                        }
                    }
                }
                stage('Lint') {
                    steps {
                        dir("${APP_DIR}") {
                            sh 'npm run lint || true'
                        }
                    }
                }
            }
        }
        
        stage('Docker Build & Deploy') {
            agent any
            stages {
                stage('Build Image') {
                    steps {
                        checkout scm
                        withCredentials([usernamePassword(
                            credentialsId: 'dockerhub-creds',
                            usernameVariable: 'DOCKER_USER',
                            passwordVariable: 'DOCKER_PASS'
                        )]) {
                            sh '''
                                echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            '''
                            sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ./Chatbot-UI"
                            sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
                        }
                    }
                }
                
                stage('Push to Docker Hub') {
                    steps {
                        sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                        sh "docker push ${IMAGE_NAME}:latest"
                    }
                }
                
                stage('Deploy to Container') {
                    steps {
                        sh 'docker stop chatbot || true'
                        sh 'docker rm chatbot || true'
                        sh """
                            docker run -d \
                                --name chatbot \
                                --restart unless-stopped \
                                -p 3000:3000 \
                                -e NODE_ENV=production \
                                ${IMAGE_NAME}:${IMAGE_TAG}
                        """
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo "Pipeline SUCCESS - image ${IMAGE_NAME}:${IMAGE_TAG} deployed"
        }
        failure {
            echo "Pipeline FAILED"
        }
        always {
            cleanWs()
        }
    }
}
