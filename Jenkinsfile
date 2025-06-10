pipeline {
    agent any
    environment {
        BRANCH_NAME = env.BRANCH_NAME
        NODE_VERSION = 'NodeJS 7.8.0'
        PORT = BRANCH_NAME == 'main' ? '3000' : '3001'
        IMAGE_NAME = BRANCH_NAME == 'main' ? 'nodemain:v1.0' : 'nodedev:v1.0'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Install Dependencies') {
            steps {
                script {
                    nodejs(NODE_VERSION) {
                        sh 'npm install'
                    }
                }
            }
        }
        stage('Run Tests') {
            steps {
                script {
                    nodejs(NODE_VERSION) {
                        sh 'npm test'
                    }
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME} ."
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    // Stop and remove existing containers
                    sh 'docker stop $(docker ps -q --filter "ancestor=${IMAGE_NAME}") || true'
                    sh 'docker rm $(docker ps -a -q --filter "ancestor=${IMAGE_NAME}") || true'

                    // Run the new container
                    sh "docker run -d --expose ${PORT} -p ${PORT}:${PORT} ${IMAGE_NAME}"
                }
            }
        }
    }
    post {
        success {
            echo '✅ Deployment successful!'
        }
        failure {
            echo '❌ Deployment failed!'
        }
    }
}
