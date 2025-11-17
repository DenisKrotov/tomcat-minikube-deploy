pipeline {
    agent any
    
    environment {
        KUBECONFIG = credentials('minikube-kubeconfig')
        GITHUB_SECRET = credentials('github-webhook-secret')
    }
    
    stages {
        stage('Setup Kubernetes Tools') {
            steps {
                script {
                    echo 'Installing kubectl...'
                    // Установка kubectl
                    sh '''
                        curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                        install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
                        kubectl version --client
                    '''
                }
            }
        }
        
        stage('Checkout Code') {
            steps {
                echo 'Checkout Code from GitHub...'
                git branch: 'main',
                    url: 'https://github.com/DenisKrotov/tomcat-minikube-deploy.git'
            }
        }
        
        stage('Deploy to Minikube') {
            steps {
                script {
                    echo 'Deploy Tomcat in Minikube...'
                    sh 'kubectl apply -f k8s/'
                    sh 'kubectl rollout status deployment/tomcat-deployment --timeout=180s'
                    echo 'Containers created'
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                script {
                    echo 'Check Deploy'
                    sh '''
                        kubectl get pods -o wide
                        kubectl get services
                        kubectl get deployments
                    '''
                }
            }
        }
    }
    
    post {
        success {
            echo '!ALL GOOD!'
            sh '''
                echo "Service created on:"
                kubectl get service tomcat-service -o jsonpath="{.status.loadBalancer.ingress[0].ip}" || echo "Use NodePort"
            '''
        }
        failure {
            echo '!ERROR!'
        }
    }
}