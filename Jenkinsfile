pipeline {
    agent any

    tools {
        sonarRunner 'SonarScanner'
    }

    environment {
        IMAGE_NAME = "deepakramamoorthytesh/flask-demo"
    }

    stages {

        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Checkout') {
            steps {
                echo "Repository Checked Out"
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'pip install -r requirements.txt'
            }
        }

        stage('Run Tests') {
            steps {
                bat 'pytest'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    bat '''
                    sonar-scanner ^
                    -Dsonar.projectKey=jenkins-python-cicd ^
                    -Dsonar.sources=. ^
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t %IMAGE_NAME% ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat """
                    docker login -u %DOCKER_USER% -p %DOCKER_PASS%
                    docker push %IMAGE_NAME%
                    """
                }
            }
        }

        stage('Deploy Container') {
            steps {
                bat '''
                docker stop flask-container || exit 0
                docker rm flask-container || exit 0
                docker run -d --name flask-container -p 5000:5000 %IMAGE_NAME%
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline Completed Successfully!'
        }

        failure {
            echo 'Pipeline Failed!'
        }
    }
}