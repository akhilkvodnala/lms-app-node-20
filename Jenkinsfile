pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'lms'  // Name of the SonarQube configuration in Jenkins
        FRONTEND_IMAGE = "akhilvodnala/frontend:latest"  // Replace with your Docker Hub username or registry
        BACKEND_IMAGE = "akhilvodnala/backend:latest"    // Replace with your Docker Hub username or registry
        AWS_REGION = "us-east-1"  // AWS region for ECR
        ECR_REPO = "982534383314.dkr.ecr.us-east-1.amazonaws.com"  // Replace with your ECR repository URL
        FRONTEND_ECR_IMAGE = "${ECR_REPO}/frontend:latest"  // Frontend ECR image
        BACKEND_ECR_IMAGE = "${ECR_REPO}/backend:latest"    // Backend ECR image
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
                    sh "docker build -t ${FRONTEND_IMAGE} ./webapp"
                    sh "docker build -t ${BACKEND_IMAGE} ./api"
                }
            }
        }

        stage('Scan Docker Images with Trivy') {
            steps {
                script {
                    def frontendScanResult = sh(script: "trivy image --no-progress --exit-code 1 ${FRONTEND_IMAGE}", returnStatus: true)
                    def backendScanResult = sh(script: "trivy image --no-progress --exit-code 1 ${BACKEND_IMAGE}", returnStatus: true)

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

                    echo "Pipeline will continue even with vulnerabilities."
                }
            }
        }

        stage('Push to AWS ECR') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-credentials-id', usernameVariable: 'AKIA6JQ4443JJWWJCGNV', passwordVariable: '4/81IhZPMXwUViqkVZvkW3ZgoPwzqMLF1xS6NH8u')]) {
                    sh '''
                    aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                    aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}
                    docker tag ${FRONTEND_IMAGE} ${FRONTEND_ECR_IMAGE}
                    docker tag ${BACKEND_IMAGE} ${BACKEND_ECR_IMAGE}
                    docker push ${FRONTEND_ECR_IMAGE}
                    docker push ${BACKEND_ECR_IMAGE}
                    '''
                }
            }
        }
    }
}
