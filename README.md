# Dockerized 1-Tier Application Deployment

This project demonstrates the end-to-end deployment of a one-tier application using Docker, CI/CD with Jenkins, and several essential security and scanning tools like SonarQube, OWASP Dependency Check, and Trivy. The infrastructure is hosted on AWS using an EC2 instance configured with the necessary software and plugins for a seamless deployment pipeline.

## **Technologies Used**
- **Git**: Version control and source code management.
- **Jenkins**: Continuous integration server for automated builds and deployments.
- **Docker**: Containerization of the application.
- **SonarQube**: Code quality and security scanning.
- **OWASP Dependency Check**: For identifying vulnerabilities in project dependencies.
- **Trivy**: Container and filesystem vulnerability scanning tool.

## **AWS EC2 Configuration**
- **AMI**: Amazon Linux 2 Kernel 5.10 AMI 2.0.20241001.0 x86_64 HVM
- **Instance Type**: t3.large
- **Security Group**: DevOps (Allowing all traffic)
- **Storage**: 25 GiB

---

## **Setup Instructions**

### 1. **EC2 Setup:**
Launch an EC2 instance with the above configurations. Once the instance is up, connect to it and install Jenkins with the following commands:


# Install Java 11 for Jenkins dependency
```bash
sudo amazon-linux-extras install java-openjdk11 -y
```
# Add Jenkins repository and install Jenkins
```
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum install jenkins -y
sudo systemctl start jenkins
```

# 2. **Install Docker:**
```
sudo yum install docker -y
sudo systemctl start docker
```
# 3. **Install Git:**
```
sudo yum install git -y
```
# 4. **SonarQube Setup:**
- **Run SonarQube using Docker:**
```
docker run --name sonarqube-custom -p 9000:9000 sonarqube:10.6-community
```

# 5. **Jenkins Configuration**

- **Install the following Jenkins plugins:**
  - Pipeline Stage View
  - Docker Pipeline
  - SonarQube Scanner
  - Node.js
  - OWASP Dependency Check
  - Eclipse Temurin Installer

- **Configure Jenkins tools and credentials for:**
  - SonarQube (Credentials for SonarQube scanning)
  - Docker (DockerHub credentials)
  - JDK 17
  - Node.js 16
  - OWASP Dependency Check


# 6. **Jenkins Pipeline Script**

  ```
  pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'nodejs16'
    }
    environment {
        SCANNER_HOME = tool 'mysonar'
    }
    stages {
        stage("Getting Code") {
            steps {
                git 'https://github.com/suneelprojects/docker-project.git'
            }
        }
        stage("SonarQube Checking") {
            steps {
                withSonarQubeEnv('mysonar') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=zomato \
                    -Dsonar.projectKey=zomato'''
                }
            }
        }
        stage("Installing Dependencies") {
            steps {
                sh 'npm install'
            }
        }
        stage("OWASP Dependency Check") {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'dp-check'
                dependencyCheckPublisher pattern: '**/dependency-report.xml'
            }
        }
        stage("Trivy Scanning") {
            steps {
                sh 'trivy fs . > trivy.txt'
            }
        }
        stage("Build Docker Image") {
            steps {
                sh 'docker build -t app .'
            }
        }
        stage("Docker Push") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'Docker-id') {
                        sh 'docker tag app manikantabobbili/dockerimage:docker'
                        sh 'docker push manikantabobbili/dockerimage:docker'
                    }
                }
            }
        }
        stage("Trivy Image Scan") {
            steps {
                sh 'trivy image manikantabobbili/dockerimage:docker'
            }
        }
        stage("Deploy Application") {
            steps {
                sh 'docker run -itd --name application -p 3000:3000 manikantabobbili/dockerimage:docker'
            }
        }
    }
}
```
## **Project Architecture**

![image](https://github.com/user-attachments/assets/6f2d9ce2-774b-45e2-a62d-943a9795613d)


## **Working of Pipeline**
<img width="644" alt="{6F313DE6-D214-4CA7-B6A2-792FED0F60C8}" src="https://github.com/user-attachments/assets/531f336e-aa0a-46cd-ab3c-791e8bfb34b1">


