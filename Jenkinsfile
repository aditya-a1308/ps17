pipeline {
    agent any
    environment {
        DOCKERHUB_CREDS = credentials('dockerhub-creds')     // Username+Password credential
        IMAGE_NAME      = "adityaa1308/cicd-flask-app"
        IMAGE_TAG       = "${BUILD_NUMBER}"
        EC2_HOST        = "ec2-user@15.252.137.210"
    }
    stages {
        stage('Checkout') {
            steps { checkout scm }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG -t $IMAGE_NAME:latest app/'
            }
        }
        stage('Push to Docker Hub') {
            steps {
                sh 'echo $DOCKERHUB_CREDS_PSW | docker login -u $DOCKERHUB_CREDS_USR --password-stdin'
                sh 'docker push $IMAGE_NAME:$IMAGE_TAG'
                sh 'docker push $IMAGE_NAME:latest'
            }
        }
        stage('Deploy on EC2') {
            steps {
                sshagent(credentials: ['ec2-ssh-key']) {        // SSH Username with private key credential
                    sh """
                      ssh -o StrictHostKeyChecking=no $EC2_HOST '
                        docker pull $IMAGE_NAME:$IMAGE_TAG &&
                        docker rm -f webapp || true &&
                        docker run -d --name webapp --restart unless-stopped -p 80:5000 $IMAGE_NAME:$IMAGE_TAG
                      '
                    """
                }
            }
        }
        stage('Smoke Test') {
            steps { sh 'sleep 5; curl -f http://15.252.137.210 | grep -i "version"' }
        }
    }
    post {
        always  { sh 'docker logout || true' }
        success { echo "Deployed build ${BUILD_NUMBER}" }
    }
}
