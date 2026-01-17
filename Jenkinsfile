pipeline {
    agent any

    environment {
        IMAGE_NAME = "ayeshasiddiqua1/arcora-web"
        CONTAINER_NAME = "arcora-prod"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$BUILD_NUMBER .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $IMAGE_NAME:$BUILD_NUMBER
                    '''
                }
            }
        }
         stage('Deploy Container') {
            steps {
                sh '''
                docker stop $CONTAINER_NAME || true
                docker rm $CONTAINER_NAME || true
                docker pull $IMAGE_NAME:$BUILD_NUMBER
                docker run -d -p 5001:5000 --name $CONTAINER_NAME $IMAGE_NAME:$BUILD_NUMBER
                '''
            }
        }
    }

    post {
        success {
            echo "CI pipeline completed successfully"
        }
        failure {
            echo "CI pipeline failed"
        }
    }
}
