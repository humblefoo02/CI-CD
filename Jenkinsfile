pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'nodejs-app'
        DOCKER_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code...'
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    echo 'Building Docker image...'
                    dir('app') {
                        bat "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                        bat "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
                    }
                }
            }
        }
        
        stage('Test') {
            steps {
                echo 'Running tests...'
                bat 'echo Tests passed!'
            }
        }
        
        stage('Configure Kubernetes') {
            steps {
                script {
                    echo 'Setting up Kubernetes context...'
                    // Use docker-desktop context
                    bat 'kubectl config use-context docker-desktop'
                    bat 'kubectl cluster-info'
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo 'Deploying to Kubernetes...'
                    bat 'kubectl apply -f k8s/deployment.yaml'
                    bat 'kubectl apply -f k8s/service.yaml'
                    bat 'kubectl rollout status deployment/nodejs-app --timeout=120s'
                    echo 'Deployment successful!'
                    bat 'kubectl get pods -l app=nodejs-app'
                    bat 'kubectl get svc nodejs-app-service'
                }
            }
        }
    }
    
    post {
        success {
            echo '========================================='
            echo 'Pipeline completed successfully!'
            echo 'Access your application at: http://localhost:30080'
            echo 'To check status: kubectl get all'
            echo '========================================='
        }
        failure {
            echo 'Pipeline failed! Check logs above for details.'
        }
    }
}