pipeline {
    agent any

    environment {
        DOCKER_IMAGE   = "danielrivni/my-flask-app"
        K8S_DEPLOYMENT = "my-flask-app"
        K8S_NAMESPACE  = "default"
    }

    stages {
        stage('Install Jinja2 CLI') {
            steps {
                sh 'pip install --user jinja2-cli'
            }
        }

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/DanielRivni/Automated-CI-CD-Pipeline.git'
            }
        }

        stage('Determine Version') {
            steps {
                script {
                    // 1. Identify the current branch name
                    def branchName = sh(script: "git rev-parse --abbrev-ref HEAD", returnStdout: true).trim()
                    echo "Branch name: ${branchName}"

                    // 2. If NOT 'main', use a dev version: 0.0.0-<shortCommit>
                    if (branchName != 'main') {
                        def shortCommit = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                        env.IMAGE_TAG = "0.0.0-${shortCommit}"

                    } else {
                        // 3. If branch == main, get the last annotated tag.
                        //    Then increment the part after the dash.
                        def lastTag = sh(script: "git describe --tags --abbrev=0", returnStdout: true).trim()
                        echo "Last tag on main: ${lastTag}"

                        def parts = lastTag.split('-')
                        def baseVersion = parts[0]
                        def lastBuildNumber = 0

                        if (parts.size() > 1) {
                            lastBuildNumber = parts[1].toInteger()
                        }

                        def newBuildNumber = lastBuildNumber + 1

                        env.IMAGE_TAG = "${baseVersion}-${newBuildNumber}"
                    }

                    echo "Using image tag: ${env.IMAGE_TAG}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${env.DOCKER_IMAGE}:${env.IMAGE_TAG} ."
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withDockerRegistry(credentialsId: 'docker-hub-credentials', url: 'https://index.docker.io/v1/') {
                    sh "docker push ${env.DOCKER_IMAGE}:${env.IMAGE_TAG}"
                }
            }
        }

        stage('Render Jinja2 Template') {
            steps {
                script {
                    sh """
                      jinja2 deployment.yaml.j2 --format=yaml \\
                        -D docker_image=${env.DOCKER_IMAGE} \\
                        -D image_tag=${env.IMAGE_TAG} > generated-deployment.yaml
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh "kubectl -n ${env.K8S_NAMESPACE} apply -f generated-deployment.yaml -f service.yaml --wait=true"
                }
            }
        }
    }
}
