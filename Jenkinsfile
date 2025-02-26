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
                    sh 'docker build -t $DOCKER_IMAGE:$IMAGE_TAG .'
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withDockerRegistry(credentialsId: 'docker-hub-credentials', url: 'https://index.docker.io/v1/') {
                    sh 'docker push $DOCKER_IMAGE:$IMAGE_TAG'
                }
            }
        }

stage('Generate Kubernetes Deployment File') {
    steps {
        script {
            def deploymentFileContent = """
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-flask-app
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "5000"
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-flask-app
  template:
    metadata:
      labels:
        app: my-flask-app
    spec:
      containers:
      - name: my-flask-app
        image: ${DOCKER_IMAGE}:${IMAGE_TAG}  # Dynamically use the image tag
        imagePullPolicy: Always
        ports:
        - containerPort: 5000
            """
            writeFile(file: 'generated-deployment.yaml', text: deploymentFileContent)
        }
    }
}


        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl -n $K8S_NAMESPACE apply -f generated-deployment.yaml -f service.yaml --wait=true'
            }
        }
    }
}
