pipeline {
    agent {
      label 'builtin_node'
    }
    parameters {
        string(name: 'DOCKER_TAG', description: 'Enter the tag for the Docker image', defaultValue: 'latest')
    }

    environment {
        SONARQUBE_TOKEN = credentials('sonarqube-token')
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the Git repository containing the Dockerfile
                git branch: 'main',
                  url: 'https://github.com/networknuts/jenkins-docker-project.git'
            }
        }

        stage('Scan Dockerfile with SonarQube') {
            steps {
                script {
                    // Assuming you have SonarQube scanner configured in Jenkins
                    def scannerHome = tool 'SonarQubeScanner'
                    withSonarQubeEnv('SonarQubeScanner') {
                        sh "/usr/local/bin/sonar-scanner -Dsonar.projectBaseDir=. -Dsonar.sources=. -Dsonar.host.url=192.168.1.11:9000 -Dsonar.login=${env.SONARQUBE_TOKEN}"
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Make sure Docker is installed and configured properly on Jenkins
                    def dockerImage = docker.build("python-jenkins-app:${params.DOCKER_TAG}", "-f Dockerfile .")
                    // You can add any additional build arguments if needed
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    // Make sure you have DockerHub credentials configured in Jenkins
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials') {
                        // Push the built Docker image to Docker Hub
                        dockerImage.push("${params.DOCKER_TAG}")
                    }
                }
            }
        }
    }
}
