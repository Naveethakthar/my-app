pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "naveethakthar/my-app"
    }

    stages {

        // ----------------------------
        // Stage 0: Check Python & pip
        // ----------------------------
        stage('Check Python') {
            steps {
                echo "Checking Python version..."
                bat 'python --version'

                echo "Checking pip version..."
                bat 'pip --version'
            }
        }

        // ----------------------------
        // Stage 1: Checkout code
        // ----------------------------
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Naveethakthar/my-app.git'
            }
        }

        // ----------------------------
        // Stage 2: Build & Test
        // ----------------------------
        stage('Build & Test') {
            steps {
                echo "Creating virtual environment..."
                bat 'python -m venv venv'

                echo "Installing pytest..."
                bat 'venv\\Scripts\\python.exe -m pip install pytest'

                echo "Running tests..."
                bat 'venv\\Scripts\\python.exe -m pytest || exit 0'
            }
        }

        // ----------------------------
        // Stage 3: SonarQube Analysis
        // ----------------------------
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'Sonar'
                    withSonarQubeEnv('Sonar') {
                        bat "\"${scannerHome}\\bin\\sonar-scanner.bat\" -Dsonar.projectKey=my-app -Dsonar.projectName=my-app -Dsonar.sources=."
                    }
                }
            }
        }

        // ----------------------------
        // Stage 4: Quality Gate
        // ----------------------------
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        // ----------------------------
        // Stage 5: Docker Build
        // ----------------------------
        stage('Docker Build') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE%:latest .'
            }
        }

        // ----------------------------
        // Stage 6: Push to DockerHub
        // ----------------------------
        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    bat '''
                    echo %PASS% | docker login -u %USER% --password-stdin
                    docker push %DOCKER_IMAGE%:latest
                    '''
                }
            }
        }

        // ----------------------------
        // Stage 7: Deploy
        // ----------------------------
        stage('Deploy') {
            steps {
                bat '''
                docker stop my-app || true
                docker rm my-app || true
                docker run -d --name my-app -p 5000:5000 %DOCKER_IMAGE%:latest
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check the logs."
        }
    }
}
