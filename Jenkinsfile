pipeline {

    agent {
        node {
            label 'roboshop'
        }
    }

    environment {
        appVersion = ''
    }

    options {
        timeout(time: 5, unit: 'MINUTES')
    }

    stages {

        stage('Read version'){
            steps {
                script {
                    // Load and parse the JSON file
                    def packageJson = readJSON file: 'package.json'
                    
                    // Access fields directly
                    appVersion = packageJson.version
                    echo "Building version ${appVersion}"
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    npm install
                '''
            }
        }

        stage('Build Image') {
            steps {
                sh """
                    sudo docker build -t catalogue:${appVersion} .
                """
            }
        }
    }
}