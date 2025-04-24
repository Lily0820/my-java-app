pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('dockerhub-creds') // Jenkins credential ID
        IMAGE_NAME = 'likitha0820/my-java-app' // Your DockerHub repo name
    }

    tools {
        maven 'maven3'
        jdk 'jdk17'
    }

    stages {
        stage('Clone Code') {
            steps {
                git url: 'https://github.com/Lily0820/my-java-rest-api.git', branch: 'main'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh 'echo $PASSWORD | docker login -u $USERNAME --password-stdin'
                    sh 'docker push $IMAGE_NAME'
                }
            }
        }

        stage('Deploy to AWS EC2') {
            steps {
                sshagent (credentials: ['ec2-key']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ec2-user@<EC2_PUBLIC_IP> '
                    docker pull $IMAGE_NAME &&
                    docker stop myapp || true &&
                    docker rm myapp || true &&
                    docker run -d -p 8081:8080 --name myapp $IMAGE_NAME'
                    '''
                }
            }
        }
    }
}
