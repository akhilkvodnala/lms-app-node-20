pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'lms'  // Name of the SonarQube configuration in Jenkins
        FRONTEND_IMAGE = "akhilvodnala/frontend:latest"  // Replace with your Docker Hub username or registry
        BACKEND_IMAGE = "akhilvodnala/backend:latest"    // Replace with your Docker Hub username or registry
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
                    script {
                        def scannerHome = tool name: 'SonarScanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=lms -Dsonar.sources=."
                    }
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    // Build the frontend Docker image from the webapp directory
                    sh "docker build -t ${FRONTEND_IMAGE} ./webapp"

                    // Build the backend Docker image from the api directory
                    sh "docker build -t ${BACKEND_IMAGE} ./api"
                }
            }
        }

        stage('Scan Docker Images with Trivy') {
            steps {
                script {
                    // Run Trivy scan for frontend and backend images
                    sh """
                    trivy image --no-progress --exit-code 1 ${FRONTEND_IMAGE}
                    trivy image --no-progress --exit-code 1 ${BACKEND_IMAGE}
                    """
                }
            }
        }
    }
}

