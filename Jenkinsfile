pipeline {
    agent any
    parameters {
        string(name: 'DOCKER_TAG', description: 'Enter the tag for the Docker image', defaultValue: 'latest')
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the Git repository containing the Dockerfile
                git branch: 'main',
                  url: 'https://github.com/networknuts/jenkins-docker-project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Make sure Docker is installed and configured properly on Jenkins
                    def dockerImage = docker.build("python-jenkins-app:${params.DOCKER_TAG}", "-f Dockerfile .")
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials') {
                                        dockerImage.push("${params.DOCKER_TAG}")
                    }
                    // You can add any additional build arguments if needed
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    // Make sure you have DockerHub credentials configured in Jenkins
                    docker.withRegistry('', 'dockerhub-credentials') {
                        // Push the built Docker image to Docker Hub
                        dockerImage.push("${params.DOCKER_TAG}")
                    }
                }
            }
        }
    }
}
