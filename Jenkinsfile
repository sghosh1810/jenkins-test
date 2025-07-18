pipeline {
    agent any

    environment {
        IMAGE_NAME = "sghosh1403/jenkins-test"
        REMOTE_HOST = "64.227.149.222"
        REMOTE_USER = "root"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/sghosh1810/jenkins-test.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}:latest")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withDockerRegistry([credentialsId: 'docker-hub-creds', url: '']) {
                    script {
                        docker.image("${IMAGE_NAME}:latest").push()
                    }
                }
            }
        }

        stage('Deploy to Server') {
            steps {
                sshagent(['ssh-remote-creds']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} '
                        docker pull ${IMAGE_NAME}:latest &&
                        docker stop express_app || true &&
                        docker rm express_app || true &&
                        docker run -d --name express_app -p 3000:3000 ${IMAGE_NAME}:latest
                    '
                    """
                }
            }
        }
    }
}
