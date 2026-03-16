pipeline {
    agent any

    environment {
        CONTAINER_NAME = "springboot-container"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/master2']],
                    extensions: [],
                    userRemoteConfigs: [[
                        credentialsId: 'git-creds',
                        url: 'https://github.com/BadamTeja/E-commerce-project-springBoot.git'
                    ]]
                )
            }
        }

        stage('Build Application') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Docker Build, Login & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    IMAGE_NAME=$DOCKER_USER/springboot-app:${BUILD_NUMBER}
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker build -t $IMAGE_NAME .
                    docker push $IMAGE_NAME
                    '''
                }
            }
        }

        stage('Deploy Container') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    IMAGE_NAME=$DOCKER_USER/springboot-app:${BUILD_NUMBER}
                    docker stop springboot-container || true
                    docker rm springboot-container || true
                    docker run -d -p 8082:8080 --name springboot-container $IMAGE_NAME
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Build & Deployment Successful!'
        }
        failure {
            echo '❌ Pipeline Failed!'
        }
    }
}
