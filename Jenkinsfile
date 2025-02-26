pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "danielrivni/my-flask-app"
        K8S_DEPLOYMENT = "my-flask-app"
        K8S_NAMESPACE = "default"
        IMAGE_TAG = "${GIT_COMMIT.take(7)}"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/DanielRivni/Automated-CI-CD-Pipeline.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-hub-credentials', url: 'https://index.docker.io/v1/') {
                        docker.image("${DOCKER_IMAGE}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Generate Deployment and Service Files') {
            steps {
                script {
                    // Replace the image tag in the deployment.yaml and service.yaml files with the Git commit hash
                    sh """
                    sed 's|danielrivni/my-flask-app:latest|${DOCKER_IMAGE}:${IMAGE_TAG}|g' deployment.yaml > deployment-temp.yaml
                    mv deployment-temp.yaml deployment.yaml
                    
                    sed 's|danielrivni/my-flask-app:latest|${DOCKER_IMAGE}:${IMAGE_TAG}|g' service.yaml > service-temp.yaml
                    mv service-temp.yaml service.yaml
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh "kubectl -n $K8S_NAMESPACE apply -f deployment.yaml -f service.yaml --wait=true"
                }
            }
        }
    }
}
