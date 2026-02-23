pipeline {
    agent any

    environment {
        IMAGE_NAME = "srinion/jenkins-demo"
        CONTAINER_NAME = "jenkins-demo-app"
        HOST_PORT = "8090"   // change if needed
        CONTAINER_PORT = "8080"
    }

    tools {
        maven 'Maven3'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/SriniGovinda/jenkins-demo.git'
            }
        }

        stage('Build JAR') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t $IMAGE_NAME:${BUILD_NUMBER} .
                docker tag $IMAGE_NAME:${BUILD_NUMBER} $IMAGE_NAME:latest
                """
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Push Image') {
            steps {
                sh """
                docker push $IMAGE_NAME:${BUILD_NUMBER}
                docker push $IMAGE_NAME:latest
                """
            }
        }

        stage('Deploy') {
            steps {
                sh """
                docker rm -f $CONTAINER_NAME || true
                docker run -d \
                  --name $CONTAINER_NAME \
                  --restart unless-stopped \
                  -p $HOST_PORT:$CONTAINER_PORT \
                  $IMAGE_NAME:latest
                """
            }
        }

        stage('Show Running Containers') {
            steps {
                sh 'docker ps'
            }
        }
    }
}