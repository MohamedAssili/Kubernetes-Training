def commit_id
pipeline {
    agent any


    stages {
        stage('Preparation') {
            steps {
                // Checkout code from SCM
                checkout scm
		sh "git rev-parse --short HEAD > .git/commit-id"
                // Get the latest commit ID and save it to a file
                script {
                    commit_id = reeadFile('.git/commit-id').trim()
                }
            }
        }

        stage('Build') {
            steps {
                echo 'Building...'
                
                // SCP to Minikube
                sh "scp -r -i $(minikube ssh-key) ./* docker@$(minikube ip):~/"

                // Build Docker image
                sh "minikube ssh 'docker build -t webapp:${commit_idd} .'"
                
                echo 'Build complete'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying to Kubernetes'

                // Update Docker image tag in deployment.yaml
                sh "sed -i -r 's|richardchesterwood/k@s-fleetman-webapp-angular:release0-5|webapp:${commit_id}|' deployment.yaml"
                
                // Apply Kubernetes deployment
                sh "kubectl get all"
                sh "kubectl apply -f deployment.yaml"
                sh "kubectl get all"
                
                echo 'Deployment complete'
            }
        }
    }

