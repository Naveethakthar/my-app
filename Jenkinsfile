pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "naveethakthar/simple-app"
    }

    stages {

        // ✅ Step 1: Check if Python is available
        stage('Check Python') {
            steps {
                bat 'python --version'
                bat 'pip --version'
            }
        }

        // Step 2: Checkout code from Git
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Naveethakthar/my-app.git'
            }
        }

        // Step 3: Build & Test using virtual environment
        stage('Build & Test') {
            steps {
                // Create a virtual environment
                bat 'python -m venv venv'

                // Activate venv and install pytest
                bat 'call venv\\Scripts\\activate && pip install --upgrade pip && pip install pytest'

                // Run tests inside the virtual environment
                bat 'call venv\\Scripts\\activate && pytest'
            }
        }

        // Step 4: SonarQube Analysis
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    bat 'sonar-scanner -Dsonar.projectKey=simple-app -Dsonar.sources=.'
                }
            }
        }

        // Step 5: Quality Gate
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        // Step 6: Docker Build
        stage('Docker Build') {
            steps {
                bat "docker build -t %DOCKER_IMAGE%:latest ."
            }
        }

        // Step 7: Push to DockerHub
        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    bat """
                    echo %PASS% | docker login -u %USER% --password-stdin
                    docker push %DOCKER_IMAGE%:latest
                    """
                }
            }
        }

        // Step 8: Deploy Docker container
        stage('Deploy') {
            steps {
                bat """
                docker stop simple-app || echo Container not running
                docker rm simple-app || echo Container not removed
                docker run -d --name simple-app -p 5000:5000 %DOCKER_IMAGE%:latest
                """
            }
        }
    }
}
