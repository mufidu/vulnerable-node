# CI/CD Pipeline with Jenkins and SonarQube Integration

This repository implements a Continuous Integration/Continuous Deployment (CI/CD) pipeline with Static Application Security Testing (SAST) using Jenkins and SonarQube. The pipeline is designed to analyze and deploy a purposefully vulnerable Node.js application to demonstrate security scanning capabilities.

## Infrastructure Setup

### Docker Compose Configuration
The infrastructure is set up using Docker Compose, with two main services:

1. **Jenkins**
   - Uses the `jenkins/jenkins:lts` image
   - Exposed ports: 8080 (web interface), 50000 (agent connection)
   - Configured with Docker-in-Docker capability
   - Persistent data in `jenkins_home` volume

2. **SonarQube**
   - Uses the `sonarqube:lts` image
   - Exposed port: 9000
   - Configured with elasticsearch bootstrap checks disabled
   - Persistent volumes for data, extensions, and logs

### Network Configuration
- All services run on a custom bridge network named `cicd-network`
- ngrok is used as a proxy to enable GitHub webhook integration

## Pipeline Features

### Build Parameters
The pipeline supports three build types:
1. **Scan Only**: Runs SonarQube analysis without deployment
2. **Scan + Deploy**: Performs both security scanning and deployment
3. **Deploy Only**: Skips security scanning and only performs deployment

### Pipeline Stages

1. **Checkout**
   - Retrieves code from SCM

2. **SonarQube Analysis**
   - Runs when build type is 'Scan Only' or 'Scan + Deploy'
   - Performs static code analysis
   - Configures exclusions and coverage report paths

3. **Quality Gate**
   - Waits for SonarQube quality gate results
   - Aborts pipeline if quality checks fail
   - 5-minute timeout configuration

4. **Build Application**
   - Builds Docker image for the application
   - Tags images with build number and 'latest'

5. **Run Application**
   - Run application container
   - Exposes application on port 8081 (8080 is used by Jenkins)

## Security Analysis Results

### Quality Gate Status
The most recent SonarQube Quality Gate status is 'Failed', as expected due to the vulnerabilities in the application.

### Security Findings
Total of 11 Security Hotspots identified with varying priority levels:

#### High Priority
- **Authentication Issues** (2 findings)

#### Medium Priority
- **Denial of Service (DoS)** (1 finding)
- **Weak Cryptography** (6 findings)

#### Low Priority
- **Insecure Configuration** (1 finding)
- **Others** (1 finding)

### Additional Metrics
- **Code Smells**: 257
- **Technical Debt**: 3d 3h
- **Reliability**: A
- **Security**: A
- **Maintainability**: A

### Notifications
- Integrated with Discord webhook for build notifications
- Sends colorized notifications based on build status
- Includes build number, job name, and status

## Applied Security Measures in Jenkins

1. **Credential Management**
   - SonarQube token stored securely in Jenkins credentials
   - Environment variables used for sensitive data

## Prerequisites

- Docker and Docker Compose
- Jenkins with following plugins:
  - NodeJS Plugin
  - SonarQube Scanner
  - Docker Pipeline
- ngrok for webhook integration
- Discord webhook URL for notifications

## Getting Started

1. Clone the repository
2. Navigate to `.ci-cd` directory
3. Run `docker-compose up -d`
4. Access Jenkins at `http://localhost:8080`
5. Access SonarQube at `http://localhost:9000`
6. Configure Jenkins credentials and SonarQube token
7. Set up webhook in GitHub repository

## Notes

- The pipeline includes proper cleanup procedures in post-build actions
- There are 3 build parameters: 'Scan Only', 'Scan + Deploy', and 'Deploy Only'
    - 'Scan Only': Only runs SonarQube analysis (no build or run)
    - 'Scan + Deploy': Runs SonarQube analysis, builds Docker image, and runs application
    - 'Deploy Only': Skips SonarQube analysis and only builds and runs application