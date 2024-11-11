pipeline {
    agent any

    // Environment variables
    environment {
        SONAR_TOKEN = credentials('sonar-token')
    }

    tools {
        nodejs 'nodejs'
        'hudson.plugins.sonar.SonarRunnerInstallation' 'sonar-scanner'
    }

    // Parameters for build type selection
    parameters {
        choice(
            name: 'BUILD_TYPE',
            choices: ['Scan Only', 'Scan + Deploy', 'Deploy Only'],
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
            when {
                expression { params.BUILD_TYPE == 'Scan Only' || params.BUILD_TYPE == 'Scan + Deploy' }
            }
            steps {
                withSonarQubeEnv('sonar') {
                    script {
                        def scannerHome = tool 'sonar-scanner'
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=jenkins \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=http://sonarqube:9000 \
                            -Dsonar.login=${SONAR_TOKEN} \
                            -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info \
                            -Dsonar.exclusions=**/scanner/**
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            when {
                expression { params.BUILD_TYPE == 'Scan Only' || params.BUILD_TYPE == 'Scan + Deploy' }
            }
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Application') {
            when {
                expression { params.BUILD_TYPE == 'Scan + Deploy' || params.BUILD_TYPE == 'Deploy Only' }
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

        stage('Run Application') {
            when {
                expression { params.BUILD_TYPE == 'Scan + Deploy' || params.BUILD_TYPE == 'Deploy Only' }
            }
            steps {
                script {
                    // Run Docker container
                    sh """
                        docker rm -f vulnerable-app || true
                        docker run -d -p 8081:8081 --name vulnerable-app vulnerable-app:latest
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
                def status = currentBuild.result ?: 'SUCCESS'
                def color = status == 'SUCCESS' ? 3066993 : 15158332 // Green or Red
                def webhookUrl = 'https://discord.com/api/webhooks/1305384533952036956/h8LYGW3Uekkbfg9HPlIEbQJQ_GbTJkLcbuwnR5JOoBWkbd9zpiGZo5LedUaJV22ZD-vZ'
                
                // Create proper Discord webhook payload
                def payload = """
                    {
                        "embeds": [{
                            "title": "Build Status",
                            "description": "Build #${BUILD_NUMBER}",
                            "color": ${color},
                            "fields": [
                                {
                                    "name": "Status",
                                    "value": "${status}",
                                    "inline": true
                                },
                                {
                                    "name": "Job",
                                    "value": "${JOB_NAME}",
                                    "inline": true
                                }
                            ]
                        }]
                    }
                """
                
                // Send webhook using curl
                sh """
                    curl -X POST \
                        -H 'Content-Type: application/json' \
                        -d '${payload.replaceAll("'", "'\\''").trim()}' \
                        ${webhookUrl}
                """
            }
        }
    }
}