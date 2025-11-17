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
                    sh '''
                        curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                        chmod +x ./kubectl
                        mkdir -p $HOME/bin
                        mv ./kubectl $HOME/bin/
                        export PATH="$HOME/bin:$PATH"
                        kubectl version --client
                    '''
                }
            }
        }
        
        stage('Checkout Code') {
            steps {
                echo 'Checkout Code from GitHub1...'
                git branch: 'main',
                    url: 'https://github.com/DenisKrotov/tomcat-minikube-deploy.git'
            }
        }
        
        stage('Deploy to Minikube') {
            steps {
                script {
                    echo 'Deploy Tomcat in Minikube...'
                    sh '''
                        export PATH="$HOME/bin:$PATH"
                        kubectl apply -f k8s/ --validate=false
                        kubectl rollout status deployment/tomcat-deployment --timeout=180s
                    '''
                    echo 'Containers created'
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                script {
                    echo 'Check Deploy'
                    sh '''
                        export PATH="$HOME/bin:$PATH"
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
                export PATH="$HOME/bin:$PATH"
                echo "Service created on:"
                kubectl get service tomcat-service
            '''
        }
        failure {
            echo '!ERROR!'
        }
    }
}
