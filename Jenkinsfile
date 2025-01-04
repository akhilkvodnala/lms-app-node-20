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
                    def frontendScanResult = sh(script: "trivy image --no-progress --exit-code 1 ${FRONTEND_IMAGE}", returnStatus: true)
                    def backendScanResult = sh(script: "trivy image --no-progress --exit-code 1 ${BACKEND_IMAGE}", returnStatus: true)

                    // Check if any vulnerabilities were found (exit code 1) and log accordingly
                    if (frontendScanResult == 1) {
                        echo "Vulnerabilities found in frontend image."
                    } else {
                        echo "No vulnerabilities found in frontend image."
                    }

                    if (backendScanResult == 1) {
                        echo "Vulnerabilities found in backend image."
                    } else {
                        echo "No vulnerabilities found in backend image."
                    }

                    // Optionally fail the build if vulnerabilities of certain severity are found
                    if (frontendScanResult != 0 || backendScanResult != 0) {
                        error("Build failed due to vulnerabilities in Docker images.")
                    }
                }
            }
        }
    }
}
