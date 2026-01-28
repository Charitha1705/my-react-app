pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/charitha1705/my-react-app.git'
            }
        }
        stage('Build') {
            steps {
                sh 'docker build -t my-react-app:latest .'
            }
        }
        stage('Test') {
            steps {
                sh 'docker run --rm my-react-app:latest npm test -- --watchAll=false'
            }
        }
        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                    sh 'docker tag my-react-app:latest 1ms24mc020/my-react-app:latest'
                    sh 'docker push 1ms24mc020/my-react-app:latest'
                }
            }
        }
        stage('Deploy') {
            steps {
                sh 'docker stack deploy -c docker-compose.yml react-stack'
            }
        }
    }
}
