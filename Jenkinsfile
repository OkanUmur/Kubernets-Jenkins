pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "okanumur/kubernets:${BUILD_NUMBER}"
    }

    triggers {
        githubPush() // webhook tetiklemesi
    }

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/OkanUmur/Kubernets-Jenkins.git'
            }
        }

        stage('Build JAR') {
            steps {
                sh './gradlew clean build'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh "docker push ${DOCKER_IMAGE}"
            }
        }

        stage('Update Kubernetes Deployment') {
            steps {
                sh "kubectl set image deployment/kubernets-deployment kubernets-container=${DOCKER_IMAGE}"
            }
        }

        stage('Apply Service') {
            steps {
                sh "kubectl apply -f k8s/service.yaml"
            }
        }
    }
}
