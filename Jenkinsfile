pipeline {
    agent any

    environment {
        IMAGE_NAME = "okanumur/kubernets-jenkins"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
       stage('Clone Repo') {
           steps {
               git branch: 'main', url: 'https://github.com/OkanUmur/Kubernets-Jenkins.git'
           }
       }

        stage('Build with Gradle') {
            steps {
                sh './gradlew clean build -x test'
            }
        }

        stage('Build Docker Image') {
            steps {
                // TAG ekle build komutuna
                sh "docker build -t $IMAGE_NAME:$IMAGE_TAG ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    // TAG ekle push komutuna
                    sh "docker push $IMAGE_NAME:$IMAGE_TAG"
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                // deployment.yaml içindeki imajı BUILD_NUMBER ile güncelle
                sh "sed -i 's|image: okanumur/kubernets-jenkins:.*|image: $IMAGE_NAME:$IMAGE_TAG|' k8s/deployment.yaml"
                // apply deployment ve service
                sh 'kubectl apply -f k8s/deployment.yaml'
                sh 'kubectl apply -f k8s/service.yaml'
            }
        }
    }
}
