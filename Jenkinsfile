pipeline {
    agent any
    tools {
        maven "Maven-3.9.7"
    }
    environment {
        COMPOSE_FILE = 'docker-compose.yml'
        DOCKERHUB_CREDENTIALS = 'iyminds-dockerhub'
        DOCKERHUB_REPO = 'iyminds/api-gateway' // Updated Docker Hub repository name
        IMAGE_TAG = 'latest' // You can change this to your desired tag name
        GIT_CREDENTIALS_ID = 'GithubCredentials'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'GithubCredentials', url: 'https://github.com/Saoudyahya/apigateway-spring-boot']])
            }
        }

        stage('Build Docker Images') {
            steps {
                sh 'mvn clean install -DskipTests'
                sh 'docker-compose build'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIALS, usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                        echo "Retrieved Docker Hub credentials successfully."
                        sh 'echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_USERNAME --password-stdin'

                        // Tag the built image
                        sh "docker tag client-microservice:latest ${DOCKERHUB_REPO}:${IMAGE_TAG}"

                        // Push the image to Docker Hub
                        sh "docker push ${DOCKERHUB_REPO}:${IMAGE_TAG}"
                    }
                }
            }
        }
    }

    post {
        always {
            sh 'docker-compose down --remove-orphans'
        }
        success {
            echo 'Deployment was successful!'
        }
        failure {
            echo 'Deployment failed!'
            sh 'docker image prune -f'
        }
    }
}
