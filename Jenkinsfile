pipeline {
    agent any
    environment {
        KUBECONFIG = "${env.HOME}/.kube/config"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Setup Environment') {
            steps {
                sh '''
                minikube status || minikube start --driver=docker
                eval $(minikube docker-env)
                '''
            }
        }
        
        stage('Run Ansible') {
            steps {
                ansiblePlaybook(
                    playbook: 'ansible/playbook.yml',
                    inventory: 'ansible/inventory',
                    credentialsId: '',
                    disableHostKeyChecking: true
                )
            }
        }
        
        stage('Verify') {
            steps {
                sh 'minikube service hello-devops-service --url'
            }
        }
    }
    post {
        always {
            sh 'docker system prune -f || true'
        }
    }
}
