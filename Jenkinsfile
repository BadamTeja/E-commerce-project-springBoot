pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "springboot-app:${BUILD_NUMBER}"
        CONTAINER_NAME = "springboot-container"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scmGit(branches: [[name: '*/master2']], extensions: [], userRemoteConfigs: [[credentialsId: 'git-creds', url: 'https://github.com/BadamTeja/E-commerce-project-springBoot.git']])
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

        stage('Docker Build Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                sh 'docker push $DOCKER_IMAGE'
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                docker stop $CONTAINER_NAME || true
                docker rm $CONTAINER_NAME || true
                docker run -d -p 8080:8080 --name $CONTAINER_NAME $DOCKER_IMAGE
                '''
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
