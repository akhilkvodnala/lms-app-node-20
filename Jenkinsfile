pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'SonarQube'  // Must match the name in Jenkins settings
        SONAR_SCANNER = '/opt/sonar-scanner/bin/sonar-scanner'  // Define the correct path
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', credentialsId: 'githublogin', url: 'https://github.com/akhilkvodnala/lms-app-node-20.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def projects = ['api', 'webapp']
                    projects.each { project ->
                        if (fileExists("${project}")) {  // Check if the directory exists
                            withSonarQubeEnv('SonarQube') {
                                sh "${SONAR_SCANNER} -Dsonar.projectKey=${project} -Dsonar.sources=${project}"
                            }
                        } else {
                            echo "Skipping ${project}, directory does not exist"
                        }
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
            emailext(
                subject: 'SonarQube Analysis Passed',
                body: 'Frontend and Backend SonarQube analysis passed!',
                to: ['akhilkvodnala@gmail.com', 'shahidafrid366@gmail.com', 'machirajuraghavarao@gmail.com']
            )
        }
        failure {
            emailext(
                subject: 'SonarQube Analysis Failed',
                body: 'SonarQube analysis failed. Check Jenkins for details.',
                to: ['akhilkvodnala@gmail.com']
            )
        }
    }
}
