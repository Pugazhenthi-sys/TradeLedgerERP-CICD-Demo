pipeline {
    agent any

    environment {
        // CHANGE THIS to your Docker Hub username
        DOCKER_IMAGE = 'pugazhmsp/rice-traders-app:latest'
    }

    stages {
        stage('Clone Repository') {
            steps {
                // CHANGE THIS to your GitHub repo
                git branch: 'main',
                    url: 'https://github.com/Pugazhenthi-sys/TradeLedgerERP-CICD-Demo.git'
            }
        }

        stage('Build with Maven') {
            steps {
                bat 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t %DOCKER_IMAGE% ."
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat """
                        echo %DOCKER_PASS%| docker login -u %DOCKER_USER% --password-stdin
                        docker push %DOCKER_IMAGE%
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat 'kubectl apply -f deployment.yaml'
                // forces pods to pull the freshly pushed :latest image on re-runs
                bat 'kubectl rollout restart deployment/java-app'
                bat 'kubectl rollout status deployment/java-app --timeout=120s'
            }
        }
    }

    post {
        success { echo 'CI/CD Pipeline executed successfully!' }
        failure { echo 'Pipeline failed. Please check logs.' }
    }
}
