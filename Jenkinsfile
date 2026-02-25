pipeline {
    agent any

    environment {
        IMAGE_NAME = "okmithilesh/demo-app"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Get Commit SHA') {
            steps {
                script {
                    def commit = sh(
                        script: "git rev-parse --short HEAD",
                        returnStdout: true
                    ).trim()

                    if (!commit) {
                        error("Commit SHA not found.")
                    }

                    env.GIT_SHA = commit
                    echo "Git SHA: ${env.GIT_SHA}"
                }
            }
        }

        stage('Build JAR') {
            steps {
                sh 'mvn clean package -Dmaven.test.skip=true'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build \
                    -t ${IMAGE_NAME}:${GIT_SHA} \
                    -t ${IMAGE_NAME}:latest \
                    .
                """
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh """
                        echo $PASS | docker login -u $USER --password-stdin
                        docker push ${IMAGE_NAME}:${GIT_SHA}
                        docker push ${IMAGE_NAME}:latest
                    """
                }
            }
        }
    }
}
