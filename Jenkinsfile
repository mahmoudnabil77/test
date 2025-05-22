pipeline {
    agent any
    environment {
        // Default fallback Docker settings
        DOCKER_HOST = "unix:///var/run/docker.sock"
        KUBECONFIG = "${env.HOME}/.kube/config"
    }
    stages {
        stage('Verify Minikube') {
            steps {
                script {
                    // Check Minikube status and start if needed
                    sh '''
                    if
                        echo "Starting Minikube..."
                        minikube start 
                    fi
                    '''
                }
            }
        }

        stage('Configure Environment') {
            steps {
                script {
                    // Get Minikube Docker config without eval
                    def DOCKER_ENV = sh(
                        script: 'minikube docker-env --shell=bash | grep -v "^#"',
                        returnStdout: true
                    ).trim()
                    
                    // Store for later stages
                    env.DOCKER_ENV = DOCKER_ENV
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Use captured environment safely
                    withEnv(["DOCKER_CONFIG=${env.DOCKER_ENV}"]) {
                        sh '''
                        echo "Using Minikube Docker environment:"
                        echo "${DOCKER_CONFIG}"
                        export ${DOCKER_CONFIG}
                        docker build -t hello-devops .
                        '''
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                kubectl rollout status deployment/hello-devops --timeout=120s
                '''
            }
        }
    }
    post {
        always {
            // Cleanup resources
            sh '''
            docker system prune -f || true
            kubectl delete pods --field-selector=status.phase==Succeeded 2>/dev/null || true
            '''
        }
        success {
            // Get application URL
            sh '''
            echo "✅ Deployment successful!"
            minikube service hello-devops-service --url || true
            '''
        }
        failure {
            // Debugging help
            sh '''
            echo "❌ Pipeline failed - gathering diagnostics:"
            minikube status
            kubectl get pods
            kubectl describe deployment/hello-devops
            '''
        }
    }
}
