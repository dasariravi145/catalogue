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

        stage('Dependabot Security Check') {
            steps {
                withCredentials([string(
                    credentialsId: 'github-token',
                    variable: 'GITHUB_TOKEN'
                )]) {

                    sh '''
                        echo "Checking GitHub Dependabot alerts..."

                        RESPONSE=$(curl -sS -L \
                            -H "Accept: application/vnd.github+json" \
                            -H "Authorization: Bearer ${GITHUB_TOKEN}" \
                            -H "X-GitHub-Api-Version: 2026-03-10" \
                            "https://api.github.com/repos/dasariravi145/catalogue/dependabot/alerts?state=open&severity=high,critical&per_page=100")

                        echo "$RESPONSE" | jq .

                        ALERT_COUNT=$(echo "$RESPONSE" | jq 'length')

                        echo "High/Critical Dependabot alerts: ${ALERT_COUNT}"

                        if [ "$ALERT_COUNT" -gt 0 ]; then
                            echo "=========================================="
                            echo "SECURITY CHECK FAILED"
                            echo "High/Critical Dependabot vulnerabilities found."
                            echo "=========================================="

                            exit 1
                        fi

                        echo "No High/Critical Dependabot vulnerabilities found."
                    '''
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
            sh """
                aws ecr get-login-password --region ${region} | \
                docker login --username AWS --password-stdin \
                ${ACC_ID}.dkr.ecr.${region}.amazonaws.com

                docker build \
                -t ${ACC_ID}.dkr.ecr.${region}.amazonaws.com/roboshop/catalogue:${appVersion} .

                docker push \
                ${ACC_ID}.dkr.ecr.${region}.amazonaws.com/roboshop/catalogue:${appVersion}
            """
         }
            }
        }

    }
}