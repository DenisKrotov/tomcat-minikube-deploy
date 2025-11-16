pipeline {
    agent any
    
    environment {
        KUBECONFIG = credentials('minikube-kubeconfig')
        GITHUB_SECRET = credentials('github-webhook-secret')
    }
    
    triggers {
        // Будет работать с GitHub webhook + secret
        githubPush()
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                echo 'Checkout Code from GitHub...'
                git branch: 'main',
                    url: 'https://github.com/DenisKrotov/tomcat-minikube-deploy.git'
                
                // Логируем информацию о коммите который triggered сборку
                sh '''
                    echo "Build triggered by GitHub webhook"
                    echo "Commit: ${GIT_COMMIT}"
                    echo "Branch: ${GIT_BRANCH}"
                '''
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