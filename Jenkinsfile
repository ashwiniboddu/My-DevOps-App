pipeline {
    agent any

    environment {
        REGISTRY = "ashwiniboddu"
        IMAGE_NAME = "myapp"
    }

    stages {
        stage ('Checkout') {
            steps {
                checkout scm
            }
        }
        stage ('Build & Unit Test') {
            steps {
                // This script block explicitly forces Jenkins to overwrite the PATH with your Dashboard's Java21
                script {
                    sh 'chmod +x ./mvnw'
                    sh './mvnw clean package'
                }
            }
        }
        stage ('Build Docker Image') {
            steps {
                script {
                    def commitSha = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    env.IMAGE_TAG = commitSha
                    sh "docker build -t ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }
        stage ('Push Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    sh "docker push ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-cred', variable: 'KUBECONFIG')]) {
                    sh """
                        helm upgrade --install myapp ./helm/myapp \
                          --set image.repository=${REGISTRY}/${IMAGE_NAME} \
                          --set image.tag=${IMAGE_TAG} \
                          --atomic --timeout 5m
                    """
                }
            }
        }
    }
    post {
        success{
            echo 'Build has completed successfully'
        }
        failure {
            echo 'Build failed and deployment got stopped.'
        }
    }
}