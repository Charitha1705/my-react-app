pipeline {
    agent any

    environment {
        IMAGE_NAME = "charitha1705/my-react-app"
        IMAGE_TAG  = "latest"
        SERVICE_NAME = "react-app"
        DOCKERHUB_CREDENTIALS = "dockerhub"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                sh '''
                  docker run --rm \
                    -v "$PWD":/app \
                    -w /app \
                    node:18-alpine \
                    sh -c "npm install && npm test -- --watchAll=false"
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                  docker build -t $IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: DOCKERHUB_CREDENTIALS,
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                      echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                      docker push $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy to Docker Swarm') {
            steps {
                sh '''
                  if docker service ls | grep -q $SERVICE_NAME; then
                    docker service update \
                      --image $IMAGE_NAME:$IMAGE_TAG \
                      --force \
                      $SERVICE_NAME
                  else
                    docker service create \
                      --name $SERVICE_NAME \
                      --replicas 2 \
                      -p 80:80 \
                      $IMAGE_NAME:$IMAGE_TAG
                  fi
                '''
            }
        }
    }

    post {
        success {
            echo "✅ React app deployed successfully to Docker Swarm"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}
