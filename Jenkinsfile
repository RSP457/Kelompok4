pipeline {
    agent any

    environment {
        IMAGE_NAME = 'my-rest-api'
        DOCKER_REGISTRY = 'your-dockerhub-username'
        KUBE_CONFIG = credentials('kubeconfig') // Anda harus menambahkan credential ini di Jenkins
    }

    stages {
        stage('Linting') {
            steps {
                echo 'Running lint...'
                sh 'pip install flake8'
                sh 'flake8 app/'
            }
        }

        stage('Testing') {
            steps {
                echo 'Running tests...'
                sh 'pip install -r requirements.txt'
                sh 'pytest tests/'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh "docker build -t ${DOCKER_REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER} ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh "echo $PASSWORD | docker login -u $USERNAME --password-stdin"
                    sh "docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying to Kubernetes...'
                sh '''
                    mkdir -p ~/.kube
                    echo "$KUBE_CONFIG" > ~/.kube/config
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                '''
            }
        }
    }

    post {
        always {
            echo 'Cleaning up...'
        }
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
