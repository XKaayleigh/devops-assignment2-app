pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = "kaayleigh/fastapi-app"
        IMAGE_TAG        = "v1.0.${BUILD_NUMBER}"
    }

    triggers {
        githubPush()
    }

    stages {

        stage('Get Code') {
            steps {
                checkout scm
                echo "Checked out commit: ${env.GIT_COMMIT}"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_HUB_REPO}:${IMAGE_TAG} ."
            }
        }

        stage('Tag Image') {
            steps {
                sh "docker tag ${DOCKER_HUB_REPO}:${IMAGE_TAG} ${DOCKER_HUB_REPO}:latest"
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                        docker push ${DOCKER_HUB_REPO}:${IMAGE_TAG}
                        docker push ${DOCKER_HUB_REPO}:latest
                    """
                }
            }
        }

    }

    post {
        success {
            echo "CI complete. Image pushed as ${DOCKER_HUB_REPO}:${IMAGE_TAG}"
        }
        failure {
            echo "CI pipeline failed. Check logs above."
        }
        always {
            sh "docker logout"
        }
    }
}