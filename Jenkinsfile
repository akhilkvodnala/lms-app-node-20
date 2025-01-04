pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'lms'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://akhilkvodnala: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@github.com/akhilkvodnala/lms-app-node-20.git'
            }
        }

        stage('Code Scan - SonarQube') {
            steps {
                withSonarQubeEnv(SONARQUBE_SERVER) {
                    sh 'sonar-scanner -Dsonar.projectKey=lms -Dsonar.sources=.'
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                echo 'Building Docker images...'
            }
        }
    }
}

