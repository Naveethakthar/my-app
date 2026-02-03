pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "naveethakthar/simple-app"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Naveethakthar/My-app.git'
            }
        }

       stage('Build & Test') {
    steps {
        // Step 1: Create a virtual environment
        bat 'python -m venv venv'

        // Step 2: Activate venv and install pytest
        bat 'call venv\\Scripts\\activate && pip install --upgrade pip && pip install pytest'

        // Step 3: Run tests inside the virtual environment
        bat 'call venv\\Scripts\\activate && pytest'
    }
}


        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    bat '''
                    sonar-scanner \
                    -Dsonar.projectKey=simple-app \
                    -Dsonar.sources=.
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                bat 'docker build -t $DOCKER_IMAGE:latest .'
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    bat '''
                    echo $PASS | docker login -u $USER --password-stdin
                    docker push $DOCKER_IMAGE:latest
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                bat '''
                docker stop simple-app || true
                docker rm simple-app || true
                docker run -d --name simple-app -p 5000:5000 $DOCKER_IMAGE:latest
                '''
            }
        }
    }
}
