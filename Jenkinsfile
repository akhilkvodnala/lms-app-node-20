pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'SonarQube'  // Must match the name in Jenkins settings
        SONAR_SCANNER = 'SonarScanner'  // Must match the tool name in Jenkins
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/akhilkvodnala/lms-app-node-20.git'
            }
        }

        stage('SonarQube Analysis - Frontend') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube') {
                        sh "${SONAR_SCANNER} -Dsonar.projectKey=frontend -Dsonar.sources=frontend"
                    }
                }
            }
        }

        stage('SonarQube Analysis - Backend') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube') {
                        sh "${SONAR_SCANNER} -Dsonar.projectKey=backend -Dsonar.sources=backend"
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 5, unit: 'MINUTES') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Quality gate failed: ${qg.status}"
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            emailext subject: 'SonarQube Analysis Passed',
                     body: 'Frontend and Backend SonarQube analysis passed! erripukks',
                     to: 'akhilkvodnala@gmail.com','shahidafrid366@gmail.com','machirajuraghavarao@gmail.com'

            slackSend channel: '#devops-alerts',
                      message: '✅ SonarQube analysis passed! Frontend and Backend are clean.'
        }
        failure {
            emailext subject: 'SonarQube Analysis Failed',
                     body: 'SonarQube analysis failed. Check Jenkins for details.',
                     to: 'akhilkvodnala@gmail.com'

            slackSend channel: '#devops-alerts',
                      message: '❌ SonarQube analysis failed! Check Jenkins for details.'
        }
    }
}
