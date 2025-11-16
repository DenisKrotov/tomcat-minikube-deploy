pipeline {
    agent any
    
    environment {
        KUBECONFIG = credentials('minikube-kubeconfig')
    }
    
    stages {
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
                echo "Service created on"
                minikube service tomcat-service --url || true
            '''
        }
        failure {
            echo '!ERROR!'
        }
    }
}