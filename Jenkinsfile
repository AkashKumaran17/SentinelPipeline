pipeline {

    agent any

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                url: 'git@github.com:AkashKumaran17/SentinelPipeline.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t sentinel-app .'
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                docker stop sentinel-container || true
                docker rm sentinel-container || true
                docker run -d -p 5000:5000 --name sentinel-container sentinel-app
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh 'curl http://localhost:5000'
            }
        }
    }
}
