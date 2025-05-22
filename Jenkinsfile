pipeline {
    agent any
    environment {
        // Use Minikube's Docker daemon
        DOCKER_HOST = "tcp://${sh(script: 'minikube ip', returnStdout: true).trim()}:2376"
        DOCKER_TLS_VERIFY = "1"
        DOCKER_CERT_PATH = "${env.HOME}/.minikube/certs"
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/your-repo.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    // Build using Minikube's Docker
                    sh 'docker build -t hello-devops .'
                    
                    // Optional: Tag for local registry
                    sh 'docker tag hello-devops localhost:5000/hello-devops'
                    sh 'docker push localhost:5000/hello-devops'
                }
            }
        }
        stage('Deploy to Minikube') {
            steps {
                sh 'kubectl apply -f deployment.yaml'
                sh 'kubectl apply -f service.yaml'
                sh 'kubectl rollout status deployment/hello-devops --timeout=90s'
            }
        }
    }
    post {
        always {
            // Cleanup unused Docker images
            sh 'docker image prune -f'
        }
        success {
            // Get application URL
            sh 'echo "Application deployed at: $(minikube service hello-devops-service --url)"'
        }
    }
}