pipeline {
    agent any
    tools {
        maven 'Maven 3.8.1'
    }
    environment {
        DOCKER_IMAGE = "adarsh05122002/spring-petclinic:${BUILD_NUMBER}"
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/RA2211003010031/spring-petclinic.git'
            }
        }
        stage('Start Databases') {
            steps {
                // Ensure old containers are removed
                sh 'docker compose down -v || true'
                // Start databases in detached mode
                sh 'docker compose up -d'
                sh 'sleep 20'
            }
        }
        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withDockerRegistry([credentialsId: 'docker-hub-creds', url: 'https://index.docker.io/v1/']) {
                    sh 'docker push $DOCKER_IMAGE'
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig-creds']) {
                    sh """
                        # Replace the image in deployment.yaml with BUILD_NUMBER tag
                        sed -i 's|image: adarsh05122002/spring-petclinic:.*|image: $DOCKER_IMAGE|g' Deployment.yaml
                        kubectl apply -f Deployment.yaml
                        kubectl apply -f service.yaml
                        kubectl rollout status deployment/sample-app-deployment
                    """
                }
            }
        }
    }
    post {
        always {
            // Stop local docker containers
            sh 'docker compose down -v || true'
        }
    }
}