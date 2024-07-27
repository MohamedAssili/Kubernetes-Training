def commit_id
pipeline {
    agent any

    environment {
        COMMIT_ID = ''
    }

    stages {
        stage('Preparation') {
            steps {
                // Checkout code from SCM
                checkout scm

                // Get the latest commit ID and save it to a file
                script {
                    COMMIT_ID = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    writeFile file: 'git/commit-id', text: COMMIT_ID
                }
            }
        }

        stage('Build') {
            steps {
                echo 'Building...'
                
                // SCP to Minikube
                sh "scp -i $(minikube ssh-key) ./* docker@$(minikube ip):~/"

                // Build Docker image
                sh "minikube ssh 'docker build -t webapp:${COMMIT_ID} .'"
                
                echo 'Build complete'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying to Kubernetes'

                // Update Docker image tag in deployment.yaml
                sh "sed -i -r 's|richardchesterwood/k@s-fleetman-webapp-angular:.*|richardchesterwood/k@s-fleetman-webapp-angular:${COMMIT_ID}|g' deployment.yaml"
                
                // Apply Kubernetes deployment
                sh "kubectl get all"
                sh "kubectl apply -f deployment.yaml"
                sh "kubectl get all"
                
                echo 'Deployment complete'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
