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

    post {
        success {
            emailext(
                subject: "✅ Jenkins Build Success - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """<p>Great news!</p>
                         <p>Job <b>${env.JOB_NAME}</b> build <b>#${env.BUILD_NUMBER}</b> completed successfully.</p>
                         <p><a href="${env.BUILD_URL}">View Build Details</a></p>""",
                to: 'hello@shounak.me'
            )
        }
        failure {
            emailext(
                subject: "❌ Jenkins Build Failed - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """<p>Unfortunately, the build has failed.</p>
                         <p>Job <b>${env.JOB_NAME}</b> build <b>#${env.BUILD_NUMBER}</b> failed.</p>
                         <p><a href="${env.BUILD_URL}">View Build Logs</a></p>""",
                to: 'hello@shounak.me'
            )
        }
    }
}
