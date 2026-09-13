pipeline {

    agent any
    
    tools {
        jdk 'JDK-21'
        maven 'mvn'
    } 
    
    environment {
        IMAGE_NAME     = "abdelaziz225/devops-java-app"
        CONTAINER_NAME = "java-app"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/azez12/java-app-cicd-project.git'
            }
        }

        stage('Check Java') {
            steps {
                sh 'java -version'
                sh 'javac -version'
                sh 'mvn -version'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }

            post {
                success {
                    archiveArtifacts artifacts: 'target/*.jar',
                    fingerprint: true
                }
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                        -u "$DOCKER_USERNAME" \
                        --password-stdin
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build \
                    -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Docker Push') {
            steps {
                sh '''
                    docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                    docker stop ${CONTAINER_NAME} || true

                    docker rm ${CONTAINER_NAME} || true

                    docker pull ${IMAGE_NAME}:${BUILD_NUMBER}

                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p 8081:8080 \
                        ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }
    }
}