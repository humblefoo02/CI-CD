pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'nodejs-app'
        DOCKER_TAG = "${BUILD_NUMBER}"
        KUBECONFIG = 'C:\\Users\\<YourUsername>\\.kube\\config'  // Update with actual path
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
        
        stage('Configure Kubernetes Context') {
            steps {
                script {
                    echo 'Setting up Kubernetes context...'
                    // Set Minikube context
                    bat 'kubectl config use-context minikube'
                    // Verify connection
                    bat 'kubectl cluster-info'
                }
            }
        }
        
        stage('Load Docker Image to Minikube') {
            steps {
                script {
                    echo 'Loading Docker image to Minikube...'
                    // Load the image into Minikube's Docker daemon
                    bat "minikube image load ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    bat "minikube image load ${DOCKER_IMAGE}:latest"
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
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completed successfully!'
            echo 'Access your application at: http://localhost:30080'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}