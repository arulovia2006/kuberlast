
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
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'set12-token',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    powershell '''
                        if ([string]::IsNullOrWhiteSpace($env:DOCKER_PASSWORD)) {
                            throw "Docker Hub password is empty. Check Jenkins credential set12-dockerhub."
                        }

                        $env:DOCKER_PASSWORD |
                            docker login --username $env:DOCKER_USER --password-stdin

                        if ($LASTEXITCODE -ne 0) {
                            throw "Docker Hub login failed."
                        }
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                bat 'docker push %DOCKER_IMAGE%:latest'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat 'kubectl apply -f k8s/deployment.yaml'
                bat 'kubectl apply -f k8s/service.yaml'
            }
        }

        stage('Verify Deployment') {
            steps {
                bat 'kubectl rollout status deployment/%K8S_DEPLOYMENT% --timeout=120s'
                bat 'kubectl get pods -l app=set12-web'
                bat 'kubectl get service set12-web-service'
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Docker image built, pushed, and deployed!'
        }
        failure {
            echo 'FAILED: Check the Console Output for the stage error.'
        }
        always {
            echo 'Set 12 CI/CD pipeline execution finished.'
        }
    }
}
