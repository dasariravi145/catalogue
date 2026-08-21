pipeline {

    agent any

    options {
        timeout(time: 5, unit: 'MINUTES')
    }

    stages {

        stage('Read version') {
            steps {
                script {
                    def config = readJSON file: 'package.json'
                    def appVersion = config.version

                    echo "Building version ${appVersion}"
                }
            }
        }

        stage('Dependabot Security Check') {
            steps {

                withCredentials([
                    string(
                        credentialsId: 'dasariravi145',
                        variable: 'GITHUB_TOKEN'
                    )
                ]) {

                    sh '''
                        echo "=========================================="
                        echo "Checking GitHub Dependabot alerts"
                        echo "=========================================="

                        RESPONSE=$(curl -sS -L \
                            -H "Accept: application/vnd.github+json" \
                            -H "Authorization: Bearer ${GITHUB_TOKEN}" \
                            -H "X-GitHub-Api-Version: 2026-03-10" \
                            "https://api.github.com/repos/dasariravi145/catalogue/dependabot/alerts?state=open&severity=high,critical&per_page=100")

                        echo "GitHub API response received."

                        # Check whether GitHub returned an API error
                        API_MESSAGE=$(echo "$RESPONSE" | jq -r '.message // empty')

                        if [ -n "$API_MESSAGE" ]; then
                            echo "GitHub API Error: $API_MESSAGE"
                            exit 1
                        fi

                        ALERT_COUNT=$(echo "$RESPONSE" | jq 'length')

                        echo "High/Critical Dependabot alerts found: ${ALERT_COUNT}"

                        if [ "$ALERT_COUNT" -gt 0 ]; then

                            echo ""
                            echo "=========================================="
                            echo "SECURITY CHECK FAILED"
                            echo "High/Critical vulnerabilities found!"
                            echo "=========================================="

                            echo "$RESPONSE" | jq -r '
                                .[] |
                                "Alert #\\(.number) | Severity: \\(.security_advisory.severity) | Package: \\(.dependency.package.name) | \\(.security_advisory.summary)"
                            '

                            echo ""
                            echo "Pipeline stopped because High/Critical vulnerabilities exist."

                            exit 1
                        fi

                        echo ""
                        echo "=========================================="
                        echo "SECURITY CHECK PASSED"
                        echo "No High/Critical Dependabot alerts found."
                        echo "=========================================="
                    '''
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing dependencies'
            }
        }

        stage('Build Image') {
            steps {
                echo 'Building Docker image'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application'
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed'
        }

        success {
            echo 'Pipeline completed successfully'
        }

        failure {
            echo 'Pipeline failed'
        }
    }
}