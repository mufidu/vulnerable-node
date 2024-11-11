# CI/CD Pipeline with Jenkins and SonarQube Integration

This repository implements a Continuous Integration/Continuous Deployment (CI/CD) pipeline with Static Application Security Testing (SAST) using Jenkins and SonarQube. The pipeline is designed to analyze and deploy a purposefully vulnerable Node.js application forked from [cr0hn](https://github.com/cr0hn/vulnerable-node) to demonstrate security scanning capabilities.

## Prerequisites

- Docker and Docker Compose
- Jenkins
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

### SonarQube Configuration

- Create a new project in SonarQube named `jenkins`.

### Jenkins Configuration

- Install required plugins:
  - NodeJS Plugin
  - SonarQube Scanner
  - Docker Pipeline
  - Generic Webhook Trigger
- Configure SonarQube Server
![image](https://github.com/user-attachments/assets/2c20e5fe-723b-4d60-b8cf-b3bce3bcef5b)
- Configure SonarQube Scanner
![image](https://github.com/user-attachments/assets/a90f00c7-adb2-47c5-a0d7-6711ec949538)
- Install Docker client and Docker compose in Jenkins container
```bash
docker exec -it jenkins sh

# Install Docker client
curl -fsSLO https://get.docker.com/builds/Linux/x86_64/docker-17.04.0-ce.tgz \
  && tar xzvf docker-17.04.0-ce.tgz \
  && mv docker/docker /usr/local/bin \
  && rm -r docker docker-17.04.0-ce.tgz

# Install Docker Compose
curl -SL https://github.com/docker/compose/releases/download/v2.30.3/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose

# Make binaries executable
chmod +x /usr/local/bin/docker-compose
```

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

### Notes

There are 3 build parameters: 'Scan Only', 'Scan + Deploy', and 'Deploy Only'
   - 'Scan Only': Only runs SonarQube analysis (no build or run)
   - 'Scan + Deploy': Runs SonarQube analysis, builds Docker image, and runs application
   - 'Deploy Only': Skips SonarQube analysis and only builds and runs application

Example:

![image](https://github.com/user-attachments/assets/3b9fe8b7-4e9f-45e1-af77-cb1d33fd2c0d)

Build `#22` uses 'Scan + Deploy' while Build `#21` uses 'Deploy Only'.

## Security Analysis Results

![image](https://github.com/user-attachments/assets/3054aca8-4082-4c54-a300-337d63ab0e0d)

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

## Notifications
- Integrated with Discord webhook for build notifications
- Sends colorized notifications based on build status
- Includes build number, job name, and status

![image](https://github.com/user-attachments/assets/c5ff71b7-e65e-4963-bb10-90666cbc8c6e)

## Credential Management
- SonarQube token stored securely in Jenkins credentials
- Environment variables used for sensitive data
