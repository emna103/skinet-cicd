pipeline {
    agent any

    environment {
        DOCKER_USER = 'emna0201'
        IMAGE       = "${DOCKER_USER}/skinet-api"
        TAG         = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Build backend') {
            steps {
                sh 'dotnet build skinet.sln -c Release'
            }
        }

        stage('Build frontend') {
            steps {
                dir('client') {
                    sh 'npm ci'
                    sh 'npx ng build --configuration production'
                }
            }
        }

        stage('Build image') {
            steps {
                sh "docker build -f Dockerfile.api -t ${IMAGE}:${TAG} -t ${IMAGE}:latest ."
            }
        }

        stage('Push image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub',
                                                  usernameVariable: 'DU',
                                                  passwordVariable: 'DP')]) {
                    sh 'echo $DP | docker login -u $DU --password-stdin'
                    sh "docker push ${IMAGE}:${TAG}"
                }
            }
        }

        stage('Deploy') {
            steps {
                sh 'docker rm -f skinet-api || true'
                sh "docker run -d --name skinet-api -p 5001:8080 ${IMAGE}:${TAG}"
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }
    }
}
