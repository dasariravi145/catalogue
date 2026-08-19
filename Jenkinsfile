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

        stage('READ Version') {
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    env.appVersion = packageJson.version

                    echo "Building with version ${env.appVersion}"
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
                    docker build -t catalog:${env.appVersion} .
                """
            }
        }
    }
}