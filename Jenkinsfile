pipeline {
    agent any
    
    environment {
        DOCKER_HUB_USER = 'sekomos3d'
        IMAGE_NAME      = 'hello-world-app'
        IMAGE_TAG       = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker Image..."
                    dockerImage = docker.build("${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}")
                    dockerImageTagLatest = docker.build("${DOCKER_HUB_USER}/${IMAGE_NAME}:latest")
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials') {
                        echo "Pushing Image to Docker Hub..."
                        dockerImage.push("${IMAGE_TAG}")
                        dockerImageTagLatest.push('latest')
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo "Deploying to Kubernetes Cluster..."
                    sh "kubectl apply -f k8s/configmap.yaml"
                    sh "kubectl apply -f k8s/secret.yaml"
                    sh "kubectl apply -f k8s/deployment.yaml"
                    sh "kubectl apply -f k8s/service.yaml"
                    sh "kubectl apply -f k8s/hpa.yaml"
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
