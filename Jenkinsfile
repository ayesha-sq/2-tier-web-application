pipeline {
    agent any

    environment {
        IMAGE_NAME = "ayeshasiddiqua1/arcora-web"
        CONTAINER_NAME = "arcora-prod"
        DOCKER_NETWORK = "2-tier_web_application_arcora-network"
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
                withCredentials([usernamePassword(
                    credentialsId: 'mysql-creds',
                    usernameVariable: 'MYSQL_USER',
                    passwordVariable: 'MYSQL_PASS'
                )]) {
                    sh '''
                    docker stop $CONTAINER_NAME || true
                    docker rm $CONTAINER_NAME || true

                    docker pull $IMAGE_NAME:$BUILD_NUMBER

                    docker run -d \
                      --name $CONTAINER_NAME \
                      --network $DOCKER_NETWORK \
                      -e MYSQL_HOST=arcora-mysql \
                      -e MYSQL_USER=$MYSQL_USER \
                      -e MYSQL_PASSWORD=$MYSQL_PASS \
                      -e MYSQL_DB=devops \
                      -p 5001:5000 \
                      $IMAGE_NAME:$BUILD_NUMBER
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "CI/CD pipeline completed successfully"
        }
        failure {
            echo "CI/CD pipeline failed"
        }
    }
}
