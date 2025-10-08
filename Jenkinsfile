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
                sh 'docker compose down -v || true'
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
            environment {
                AWS_ACCESS_KEY_ID = credentials('aws-access-key-id')
                AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
                AWS_DEFAULT_REGION = 'ap-south-1'
            }
            steps {
                withKubeConfig([credentialsId: 'kubeconfig-creds']) {
                    sh '''
                        # Check if files exist
                        ls -la Deployment.yaml service.yaml
                        
                        # Replace the image tag
                        sed -i "s|image: adarsh05122002/spring-petclinic:.*|image: $DOCKER_IMAGE|g" Deployment.yaml
                        
                        # Show the updated file content
                        echo "=== Updated Deployment.yaml ==="
                        cat Deployment.yaml
                        
                        # Apply the manifests
                        kubectl apply -f Deployment.yaml
                        kubectl apply -f service.yaml
                        
                        # Wait for rollout
                        kubectl rollout status deployment/sample-app-deployment
                        
                        # Show deployment status
                        kubectl get pods,services
                    '''
                }
            }
        }
    }
    post {
        always {
            sh 'docker compose down -v || true'
        }
    }
}