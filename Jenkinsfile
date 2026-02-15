pipeline {
    agent any

    tools {
        nodejs 'node18'
    }

    environment {
        IMAGE_NAME = "dusky88/chatbot"
        IMAGE_TAG  = "${BUILD_NUMBER}"
        APP_DIR    = "Chatbot-UI"
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
                    sh 'npm ci'
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

        stage('Docker Build') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub-creds', toolName: 'docker') {
                        sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ./Chatbot-UI"
                        sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
                    }
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub-creds', toolName: 'docker') {
                        sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                        sh "docker push ${IMAGE_NAME}:latest"
                    }
                }
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

    post {
        success {
            echo "Pipeline SUCCESS - image ${IMAGE_NAME}:${IMAGE_TAG} deployed"
        }
        failure {
            echo "Pipeline FAILED at stage: ${currentBuild.result}"
        }
        always {
            sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true"
            cleanWs()
        }
    }
}
