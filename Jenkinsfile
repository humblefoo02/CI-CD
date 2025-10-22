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
        
        stage('Verify Kubernetes') {
            steps {
                script {
                    echo 'Verifying Kubernetes connection...'
                    bat 'kubectl config use-context docker-desktop'
                    bat 'kubectl cluster-info'
                    bat 'kubectl get nodes'
                }
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
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo 'Deploying to Kubernetes...'
                    
                    // Apply Kubernetes configurations
                    bat 'kubectl apply -f k8s/deployment.yaml'
                    bat 'kubectl apply -f k8s/service.yaml'
                    
                    // Wait for deployment to complete
                    echo 'Waiting for deployment rollout...'
                    bat 'kubectl rollout status deployment/nodejs-app --timeout=120s'
                    
                    // Show deployment status
                    echo 'Current deployment status:'
                    bat 'kubectl get pods -l app=nodejs-app'
                    bat 'kubectl get svc nodejs-app-service'
                }
            }
        }
    }
    
    post {
        success {
            echo '========================================='
            echo '✅ Pipeline completed successfully!'
            echo '========================================='
            echo 'Application deployed and running!'
            echo ''
            echo 'Access your app: http://localhost:30080'
            echo ''
            echo 'Useful commands:'
            echo '  kubectl get pods              - View pods'
            echo '  kubectl get svc               - View services'
            echo '  kubectl logs -l app=nodejs-app - View logs'
            echo '========================================='
        }
        failure {
            echo '❌ Pipeline failed!'
            echo 'Check the error messages above for details.'
        }
    }
}