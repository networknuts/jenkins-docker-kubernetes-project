pipeline {
    agent any
    parameters {
        string(name: 'DOCKER_TAG', description: 'Enter the tag for the Docker image', defaultValue: 'latest')
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the Git repository containing the Dockerfile and deployment.yaml
                git branch: 'main',
                  url: 'https://github.com/networknuts/jenkins-docker-project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Make sure Docker is installed and configured properly on Jenkins
                    def dockerImage = docker.build("docker.io/aryansr/python-jenkins-app:${params.DOCKER_TAG}", "-f Dockerfile .")
                    docker.withRegistry('', 'dockerhub-credentials') {
                        dockerImage.push("${params.DOCKER_TAG}")
                    }
                    // You can add any additional build arguments if needed
                }
            }
        }

        stage('Scan Docker Image for Vulnerabilities') {
            steps {
                sh "docker run -v /var/run/docker.sock:/var/run/docker.sock -v $HOME/Library/Caches:/root/.cache/ aquasec/trivy:0.51.1 image docker.io/aryansr/python-jenkins-app:${params.DOCKER_TAG}"
            }
        }

        stage('Update Image Tag in deployment.yaml') {
            steps {
                script {
                    // Replace the image tag in deployment.yaml with the specified DOCKER_TAG
                    sh "sed -i 's|image:.*|image: docker.io/aryansr/python-jenkins-app:${params.DOCKER_TAG}|g' deployment.yaml"
                }
            }
        }

        stage('Login to Kubernetes Cluster') {
            steps {
                script {
                    // Retrieve kubeconfig secret from Jenkins credentials
                    withCredentials([file(credentialsId: 'kubernetes-config', variable: 'KUBECONFIG_FILE')]) {
                        // Set kubeconfig environment variable
                        env.KUBECONFIG = sh(script: 'echo $KUBECONFIG_FILE', returnStdout: true).trim()
                        // Authenticate with Kubernetes cluster
                        sh 'kubectl version'
                    }
                }
            }
        }

        stage('Deploy to Kubernetes Cluster') {
            steps {
                script {
                    // Apply the modified Kubernetes deployment YAML
                    sh 'kubectl apply -f deployment.yaml'
                }
            }
        }
    }
}
