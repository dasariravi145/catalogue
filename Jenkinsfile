pipeline {

    agent {
        node {
            label 'roboshop'
        }
    }

    environment {
        appVersion = ''
        ACC_ID = "928747700481"
        region = "us-east-1"
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
                withAWS(credentials: "aws-creds", region: "${region}") {
                    // Verifies who you are logged in as
                    sh """
                       aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.${region}.amazonaws.com
                       sudo docker build -t ${ACC_ID}.dkr.ecr.${region}.amazonaws.com/roboshop/catalogue:${appVersion} .
                       sudo docker push ${ACC_ID}.dkr.ecr.${region}.amazonaws.com/roboshop/catalogue:${appVersion}

                    """
                }
            }
        }
        
    }
}