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
                // Public repo → no credentials needed
                checkout scm
            }
        }

        stage('Verify Dockerfile Path') {
              steps {
                sh '''
                  pwd
                  ls apps/frontend
                '''
              }
            }

        stage('Build Docker Image') {
            steps {
                sh """
                  docker build \
                    -t arvindan1308n/nginx-prod:4 \
                    -f apps/frontend/Dockerfile \
                    apps/frontend
                """
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                      echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                      docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                    """
                }
            }
        }

        stage('Update Kubernetes Manifest') {
            steps {
                sh """
                  sed -i 's|image:.*|image: ${IMAGE_NAME}:${BUILD_NUMBER}|' ${MANIFEST_PATH}
                """
            }
        }

        stage('Commit & Push Manifest Change') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'git-creds',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_PASS'
                )]) {
                    sh """
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
}
