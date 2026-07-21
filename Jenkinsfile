pipeline {

    agent any

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                url: 'git@github.com:AkashKumaran17/SentinelPipeline.git'
            }
        }


        stage('SonarQube Code Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                    sonar-scanner \
                    -Dsonar.projectKey=SentinelPipeline \
                    -Dsonar.sources=.
                    '''
                }
            }
        }


        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t sentinel-app .
                '''
            }
        }


        stage('Trivy Security Scan') {
            steps {
                sh '''
                trivy image \
                --severity HIGH,CRITICAL \
                --exit-code 0 \
                --format table \
                sentinel-app
                '''
            }
        }


        stage('Deploy Container') {
            steps {
                sh '''
                docker stop sentinel-container || true
                docker rm sentinel-container || true

                docker run -d \
                -p 5000:5000 \
                --name sentinel-container \
                sentinel-app
                '''
            }
        }


        stage('OWASP ZAP Security Scan') {
            steps {
                sh '''
                docker run --rm \
                --user root \
                --network host \
                -v $(pwd):/zap/wrk \
                zaproxy/zap-stable \
                zap-baseline.py \
                -t http://localhost:5000 \
                -r zap-report.html
                '''
            }
        }


        stage('Health Check') {
            steps {
                sh '''
                sleep 5
                curl -f http://localhost:5000
                '''
            }
        }

    }


    post {

        always {

            archiveArtifacts artifacts: 'zap-report.html',
            allowEmptyArchive: true

            echo 'Pipeline execution completed'
        }


        success {
            echo 'SentinelPipeline completed successfully'
        }


        failure {
            echo 'SentinelPipeline failed. Check logs.'
        }

    }

}
