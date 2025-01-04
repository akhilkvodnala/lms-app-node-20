pipeline {
    agent any
    stages {
        stage('Checkout SCM') {
            steps {
                git credentialsId: 'github-token', url: 'https://github.com/akhilkvodnala/lms-app-node-20.git'
            }
        }
    }
}

