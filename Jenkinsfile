pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'psakthiece'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'dev', url: 'https://github.com/psakthiece/Devops-build-Project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'chmod +x build.sh'
                sh './build.sh'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                        sh "docker tag myapp:latest $DOCKER_USER/dev:latest"
                        sh "docker push $DOCKER_USER/dev:latest"
                    }
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-ssh-key']) {
                    sh '''
                    # Copy both deploy.sh and docker-compose.yml to EC2
                    scp -o StrictHostKeyChecking=no deploy.sh docker-compose.yml ubuntu@18.207.130.95:/home/ubuntu/

                    # Run deploy.sh on EC2
                    ssh -o StrictHostKeyChecking=no ubuntu@18.207.130.95 "chmod +x ~/deploy.sh && ~/deploy.sh"
                    '''
                }
            }
        }

        stage('Health Check') {
            steps {
                script {
                    sh 'curl -f http://18.207.130.95 || exit 1'
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
