pipeline {
    agent any

    // Environment variables
    environment {
        SONAR_TOKEN = credentials('sonar-token')
    }

    // Parameters for build type selection
    parameters {
        choice(
            name: 'BUILD_TYPE',
            choices: ['Scan Only', 'Scan + Deploy'],
            description: 'Select the type of build to execute'
        )
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh """
                        sonar-scanner \
                        -Dsonar.projectKey=jenkins \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://localhost:9000 \
                        -Dsonar.login=$SONAR_TOKEN \
                        -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info \
                        -Dsonar.exclusions=**/scanner/**
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Application') {
            when {
                expression { params.BUILD_TYPE == 'Scan + Deploy' }
            }
            steps {
                script {
                    // Build Docker image
                    sh """
                        docker build -t vulnerable-app:${BUILD_NUMBER} .
                        docker tag vulnerable-app:${BUILD_NUMBER} vulnerable-app:latest
                    """
                }
            }
        }
    }

    post {
        always {
            // Clean workspace
            cleanWs()
            script {
                // Send build status to Discord
                def status = currentBuild.result ?: 'SUCCESS'
                def color = status == 'SUCCESS' ? 'good' : 'danger'
                def message = status == 'SUCCESS' ? 'Build successful' : 'Build failed'

                def payload = """
                    {
                        "username": "Jenkins",
                        "attachments": [
                            {
                                "color": "${color}",
                                "title": "${message}",
                                "fields": [
                                    {
                                        "title": "Build Number",
                                        "value": "${BUILD_NUMBER}",
                                        "short": true
                                    },
                                    {
                                        "title": "Status",
                                        "value": "${status}",
                                        "short": true
                                    }
                                ]
                            }
                        ]
                    }
                """

                sh """
                    curl -X POST \
                    -H 'Content-Type: application/json' \
                    -d '${payload}' \
                    https://discord.com/api/webhooks/1183663200525369434/h_erLvcDtuD_FDrKlttESq5UYKkLv3VayO1ka6J4h79HkV-t3MGti-yAfRJIEk39DeNV
                """
            }
        }
    }
}