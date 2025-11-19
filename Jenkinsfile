pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'my-cloud-app'
        CONTAINER_NAME = 'my-app'
        VPS_HOST = '103.139.193.31'
        VPS_USER = 'bnsp3'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/wildanziyad/bnsp3.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('app') {
                    sh 'docker build -t ${DOCKER_IMAGE} .'
                }
            }
        }

        stage('Test') {
            steps {
                sh 'docker run --rm ${DOCKER_IMAGE} nginx -t'
            }
        }

        stage('Deploy to VPS') {
            steps {
                sshagent(['vps-ssh-key']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${VPS_USER}@${VPS_HOST} '
                        cd bnsp3/app && \
                        docker stop ${CONTAINER_NAME} || true && \
                        docker rm ${CONTAINER_NAME} || true && \
                        docker run -d -p 80:80 --name ${CONTAINER_NAME} ${DOCKER_IMAGE}
                    '
                    """
                }
            }
        }

        stage('Cleanup') {
            steps {
                sh 'docker system prune -f'
            }
        }
    }

    post {
        success {
            emailext (
                subject: "SUCCESS: Job ${env.JOB_NAME} - Build ${env.BUILD_NUMBER}",
                body: "Build berhasil! Aplikasi telah di-deploy ke VPS.\n\nDetail:\n- Build URL: ${env.BUILD_URL}\n- Commit: ${env.GIT_COMMIT}",
                to: "admin@example.com"
            )
        }
        failure {
            emailext (
                subject: "FAILED: Job ${env.JOB_NAME} - Build ${env.BUILD_NUMBER}",
                body: "Build gagal! Silakan cek logs.\n\nDetail:\n- Build URL: ${env.BUILD_URL}",
                to: "admin@example.com"
            )
        }
    }
}