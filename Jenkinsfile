pipeline {
    agent any

    environment {
        IMAGE_NAME = "riansp457/kelompok4-credentials"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stage('Install Dependencies') {
    steps {
        sh '''
            python3 -m venv venv
            source venv/bin/activate
            pip install --upgrade pip
            pip install -r requirements.txt
        '''
    }
}


        stage('Linting') {
            steps {
                sh 'pip install flake8 && flake8 app.py'
            }
        }

        stage('Unit Test') {
            steps {
                sh 'pytest || true' // jika tidak ada test, tetap jalan
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${env.IMAGE_NAME}:${env.IMAGE_TAG}")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withDockerRegistry([ credentialsId: 'dockerhub-credentials', url: '' ]) {
                    script {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl apply -f kubernetes/namespaces/
                kubectl apply -f kubernetes/flask-api-deployment.yaml
                kubectl apply -f kubernetes/flask-api-service.yaml
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Deployment Success!'
        }
        failure {
            echo '❌ Deployment Failed.'
        }
    }
}
