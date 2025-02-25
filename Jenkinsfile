pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "danielrivni/my-flask-app"
        K8S_DEPLOYMENT = "my-flask-app"
        K8S_NAMESPACE = "default"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/DanielRivni/Automated-CI-CD-Pipeline.git'
            }
        }
        // stage('Set image version') {
        //     steps {
        //         script {
                    
        //         }
        //     }
        // }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t $DOCKER_IMAGE:latest .'
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withDockerRegistry(credentialsId: 'docker-hub-credentials', url: 'https://index.docker.io/v1/') {
                    sh 'docker push $DOCKER_IMAGE:latest'
                }
            }
        }

        // stage('Generate deployment file') {
        //     steps {
        //         }
        //     }
        // }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl -n $K8S_NAMESPACE apply -f deployment.yaml -f service.yaml --wait=true'
            }
        }
    }
}
