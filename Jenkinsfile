pipeline {
    agent any

    environment {
        IMAGE_NAME = "arvindan1308n/nginx-prod"
        MANIFEST_PATH = "manifests/deployment.yaml"
        GIT_REPO = "https://github.com/arvindan1308/CICD-kubernetes-project-for-beginner.git"
    }

    stages {

        stage('Checkout Source Code') {
            steps {
                // Pull latest code
                git branch: 'main', url: "${GIT_REPO}"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                  set -e
                  docker build \
                    -t ${IMAGE_NAME}:${BUILD_NUMBER} \
                    -f apps/frontend/Dockerfile \
                    apps/frontend
                """
            }
        }

        stage('Docker Login & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                      set -e
                      echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                      docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                    """
                }
            }
        }

        stage('Update Kubernetes Manifest') {
            steps {
                sh """
                  set -e
                  sed -i 's|image:.*|image: ${IMAGE_NAME}:${BUILD_NUMBER}|' ${MANIFEST_PATH}
                  cat ${MANIFEST_PATH}   # Optional: Verify change
                """
            }
        }

        stage('Commit & Push Manifest') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'git-creds',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_PASS'
                )]) {
                    sh """
                      set -e
                      git config user.email "jenkins@local"
                      git config user.name "jenkins"
                      git add ${MANIFEST_PATH}
                      git commit -m "Update image to ${IMAGE_NAME}:${BUILD_NUMBER}" || echo "No changes to commit"
                      git push https://${GIT_USER}:${GIT_PASS}@github.com/arvindan1308/CICD-kubernetes-project-for-beginner.git HEAD:main
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully for build #${BUILD_NUMBER}"
        }
        failure {
            echo "Pipeline failed at some stage. Please check the logs."
        }
    }
}
