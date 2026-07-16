pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "naveen93mm/jack"
        DOCKER_TAG = "${BUILD_NUMBER}"
        DEPLOY_SERVER = "3.108.197.166"
    }

    stages {

        stage('Clone Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Naveen93mm/demoada.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                """
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'Dockerhub',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {
                sh """
                docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                """
            }
        }

        stage('Deploy to VM') {
            steps {
                sshagent(['vm']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ec2-user@${DEPLOY_SERVER} '
                        sudo docker pull ${DOCKER_IMAGE}:${DOCKER_TAG}
                        sudo docker stop jack || true
                        sudo docker rm jack || true
                        sudo docker run -d -p 80:80 --name novaloc93 ${DOCKER_IMAGE}:${DOCKER_TAG}
                    '
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Deployment Successful "
        }
        failure {
            echo "Pipeline Failed check Console Output"
        }
    }
}
