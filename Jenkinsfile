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
                sh 'docker-compose down -v || true'
                sh 'docker-compose up -d'
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
                    echo "🔹 Updating image in Deployment manifest..."
                    sed -i "s|image: adarsh05122002/spring-petclinic:latest|image: $DOCKER_IMAGE|g" Deployment.yaml
                
                    echo "🔹 Applying manifests to Kubernetes..."
                    kubectl apply -f Deployment.yaml
                    kubectl apply -f service.yaml
                
                    echo "🔹 Waiting for rollout..."
                    kubectl rollout status deployment/sample-app-deployment
                
                    echo "🔹 Showing pod and service status..."
                    kubectl get pods,svc -o wide
                '''
            }
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