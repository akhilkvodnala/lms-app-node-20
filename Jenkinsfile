pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'lms'  // Name of the SonarQube configuration in Jenkins
        FRONTEND_IMAGE = "frontend:latest"  // Local tag for frontend Docker image
        BACKEND_IMAGE = "backend:latest"   // Local tag for backend Docker image
        AWS_REGION = 'us-east-1' // AWS Region
        ECR_REPO = '982534383314.dkr.ecr.us-east-1.amazonaws.com' // AWS ECR repository URL
        FRONTEND_ECR_IMAGE = "${ECR_REPO}/frontend:latest" // ECR URL for frontend image
        BACKEND_ECR_IMAGE = "${ECR_REPO}/backend:latest" // ECR URL for backend image
        KUBE_CONFIG = '/path/to/kubeconfig' // Ensure the Kubernetes config is available
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/akhilkvodnala/lms-app-node-20.git'
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

                    // Proceed with the pipeline even if vulnerabilities are found
                    echo "Pipeline will continue even with vulnerabilities."
                }
            }
        }

        stage('Push to AWS ECR') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws secret access key', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    script {
                        // Configure AWS CLI with the credentials
                        sh '''
                        aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                        aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                        aws configure set region ${AWS_REGION}
                        '''

                        // Login to AWS ECR
                        sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}"

                        // Tag the frontend image
                        sh "docker tag ${FRONTEND_IMAGE} ${FRONTEND_ECR_IMAGE}"

                        // Tag the backend image
                        sh "docker tag ${BACKEND_IMAGE} ${BACKEND_ECR_IMAGE}"

                        // Push the frontend image to ECR
                        sh "docker push ${FRONTEND_ECR_IMAGE}"

                        // Push the backend image to ECR
                        sh "docker push ${BACKEND_ECR_IMAGE}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBE_CONFIG')]) {
                    script {
                        // Apply Kubernetes manifests
                        sh '''
                        kubectl --kubeconfig=${KUBE_CONFIG} apply -f k8s/be-configmap.yml
                        kubectl --kubeconfig=${KUBE_CONFIG} apply -f k8s/be-deployment.yml
                        kubectl --kubeconfig=${KUBE_CONFIG} apply -f k8s/be-service.yml
                        kubectl --kubeconfig=${KUBE_CONFIG} apply -f k8s/fe-deployment.yml
                        kubectl --kubeconfig=${KUBE_CONFIG} apply -f k8s/fe-servicefile.yml
                        kubectl --kubeconfig=${KUBE_CONFIG} apply -f k8s/secrets.yml
                        kubectl --kubeconfig=${KUBE_CONFIG} apply -f k8s/service-account.yml
                        '''
                    }
                }
            }
        }
    }
}
