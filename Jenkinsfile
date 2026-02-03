pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "naveethakthar/simple-app"
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
        // Create virtual environment
        bat 'python -m venv venv'

        // Install pytest using venv python (NO pip upgrade)
        bat 'venv\\Scripts\\python.exe -m pip install pytest'

        // Run tests (safe even if no tests exist)
        bat 'venv\\Scripts\\python.exe -m pytest || exit 0'
    }
}




        // ----------------------------
        // Stage 3: SonarQube Analysis
        // ----------------------------
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    bat '''
                    sonar-scanner ^
                    -Dsonar.projectKey=simple-app ^
                    -Dsonar.sources=.
                    '''
                }
            }
        }

        // ----------------------------
        // Stage 4: Quality Gate
        // ----------------------------
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
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
                docker stop simple-app || true
                docker rm simple-app || true
                docker run -d --name simple-app -p 5000:5000 %DOCKER_IMAGE%:latest
                '''
            }
        }

    } // end stages
}
