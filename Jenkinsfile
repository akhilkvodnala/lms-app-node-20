pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url:'https://github.com/akhilkvodnala/lms-app-node-20.git'
            }
        }
        stage('Build Placeholder') {
            steps {
                echo 'Code checkout successful!'
            }
        }
    }
}

