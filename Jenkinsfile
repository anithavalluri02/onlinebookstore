pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/anithavalluri02/onlinebookstore.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE}:${BUILD_NUMBER} .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE}:${BUILD_NUMBER}
                        docker tag ${IMAGE}:${BUILD_NUMBER} ${IMAGE}:latest
                        docker push ${IMAGE}:latest
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sshagent(['server-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no user@your-server "
                            docker pull ${IMAGE}:latest &&
                            docker stop ${APP_NAME} || true &&
                            docker rm ${APP_NAME} || true &&
                            docker run -d --name ${APP_NAME} -p 8080:8080 ${IMAGE}:latest
                        "
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished. Check above for details."
        }
    }
}
