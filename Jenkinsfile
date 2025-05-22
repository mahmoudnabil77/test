pipeline {
    agent any
    environment {
        // Explicitly set Docker socket path
        DOCKER_HOST = "unix:///var/run/docker.sock"
        // Ensure PATH includes Docker binaries
        PATH = "/usr/bin:/usr/local/bin:$PATH"
    }
    stages {
        stage('Verify Environment') {
            steps {
                script {
                    // Check Docker access and version
                    sh '''
                    echo "### Docker Info ###"
                    docker --version || true
                    ls -l /var/run/docker.sock || true
                    groups | grep docker || echo "WARNING: User not in docker group"
                    '''
                }
            }
        }

        stage('Fix Permissions') {
            steps {
                script {
                    // Temporary permission fix (safe for CI)
                    sh '''
                    if [ ! -w /var/run/docker.sock ]; then
                        echo "Attempting to fix Docker socket permissions..."
                        sudo chmod 666 /var/run/docker.sock || echo "Failed to adjust permissions"
                    fi
                    '''
                }
            }
        }

        stage('Build and Deploy') {
            steps {
                script {
                    // Build with explicit Minikube Docker environment
                    sh '''
                    eval $(minikube docker-env) && \
                    docker build -t hello-devops .
                    '''
                    
                    // Deploy to Kubernetes
                    sh '''
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                    kubectl rollout status deployment/hello-devops --timeout=90s
                    '''
                }
            }
        }
    }
    post {
        always {
            // Cleanup resources
            sh '''
            docker system prune -f || true
            kubectl delete pods --field-selector=status.phase==Succeeded --wait=false 2>/dev/null || true
            '''
        }
        success {
            // Get application URL
            sh '''
            echo "Deployment successful!"
            minikube service hello-devops-service --url || true
            '''
        }
    }
}
