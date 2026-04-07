pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'psakthiece'
        DOCKERHUB_PASS = credentials('dockerhub-creds')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh './build.sh'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    sh "echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin"
                    if (env.BRANCH_NAME == 'dev') {
                        sh "docker tag react-app:dev $DOCKERHUB_USER/dev:latest"
                        sh "docker push $DOCKERHUB_USER/dev:latest"
                    }
                    if (env.BRANCH_NAME == 'master') {
                        sh "docker tag react-app:dev $DOCKERHUB_USER/prod:latest"
                        sh "docker push $DOCKERHUB_USER/prod:latest"
                    }
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-ssh-key']) {
                    sh 'ssh ubuntu@3.238.28.128 "cd ~/devops-build && ./deploy.sh"'
                }
            }
        }

        stage('Health Check') {
            steps {
                script {
                    retry(3) {
                        sh 'curl -f http://3.238.28.128 || exit 1'
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Deployment successful!"
        }
        failure {
            echo "Deployment failed. Check logs."
        }
    }
}

