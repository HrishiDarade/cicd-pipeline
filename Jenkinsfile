pipeline {
    agent any
    environment {
        NODE_VERSION = 'NodeJS 24.2.0'
    }
    stages {
        stage('Initialize') {
            steps {
                script {
                    // Dynamically calculate branch-specific values
                    def branchName = env.BRANCH_NAME
                    env.PORT = branchName == 'main' ? '3000' : '3001'
                    env.IMAGE_NAME = branchName == 'main' ? 'nodemain:v1.0' : 'nodedev:v1.0'

                    echo "Branch Name: ${branchName}"
                    echo "Port: ${env.PORT}"
                    echo "Image Name: ${env.IMAGE_NAME}"
                }
            }
        }
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
                    sh "docker build -t ${env.IMAGE_NAME} ."
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    // Stop and remove existing containers
                    sh 'docker stop $(docker ps -q --filter "ancestor=${env.IMAGE_NAME}") || true'
                    sh 'docker rm $(docker ps -a -q --filter "ancestor=${env.IMAGE_NAME}") || true'

                    // Run the new container
                    sh "docker run -d --expose ${env.PORT} -p ${env.PORT}:${env.PORT} ${env.IMAGE_NAME}"
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
