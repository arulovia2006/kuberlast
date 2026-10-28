
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'arulovia/set12-web'
        K8S_DEPLOYMENT = 'set12-web'
    }

    stages {

        stage('Checkout Source Code') {
            steps {
                checkout scm
                echo 'Source code checked out successfully.'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE%:latest .'
                echo 'Docker image built successfully.'
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'set12-token',
                        variable: 'DOCKER_TOKEN'
                    )
                ]) {
                    powershell '''
                        if ([string]::IsNullOrWhiteSpace($env:DOCKER_TOKEN)) {
                            throw "Docker Hub token is empty. Check Jenkins credential set12-token."
                        }

                        $env:DOCKER_TOKEN |
                            docker login --username arulovia --password-stdin

                        if ($LASTEXITCODE -ne 0) {
                            throw "Docker Hub login failed."
                        }
                    '''
                }
                echo 'Docker Hub login successful.'
            }
        }

        stage('Push Docker Image') {
            steps {
                bat 'docker push %DOCKER_IMAGE%:latest'
                echo 'Docker image pushed successfully.'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat 'kubectl apply -f k8s/deployment.yaml'
                bat 'kubectl apply -f k8s/service.yaml'
                echo 'Kubernetes resources applied successfully.'
            }
        }

        stage('Verify Deployment') {
            steps {
                bat 'kubectl rollout status deployment/%K8S_DEPLOYMENT% --timeout=120s'
                bat 'kubectl get pods -l app=set12-web'
                bat 'kubectl get service set12-web-service'
                echo 'Kubernetes deployment verified successfully.'
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Docker image built, pushed, and deployed to Kubernetes!'
        }

        failure {
            echo 'FAILED: Check the Console Output for the stage error.'
        }

        always {
            echo 'Set 12 CI/CD pipeline execution finished.'
        }
    }
}
