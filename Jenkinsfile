pipeline {
    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        DOCKER_USER = "arvindan1308n"
        IMAGE_NAME = "nginx-gitops"
        REPO_URL = "github.com/arvindan1308/CICD-kubernetes-project-for-beginner.git"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Push Docker') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'U', passwordVariable: 'P')]) {
                    sh '''
                    echo $P | docker login -u $U --password-stdin
                    docker build -t $DOCKER_USER/$IMAGE_NAME:$BUILD_NUMBER -f apps/frontend/Dockerfile apps/frontend
                    docker push $DOCKER_USER/$IMAGE_NAME:$BUILD_NUMBER
                    '''
                }
            }
        }

        stage('Update Manifest') {
            steps {
                sh """
                sed -i 's|${DOCKER_USER}/${IMAGE_NAME}:.*|${DOCKER_USER}/${IMAGE_NAME}:${BUILD_NUMBER}|' manifests/deployment.yaml
                """
            }
        }

        stage('Push Changes to Git') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-creds', usernameVariable: 'GU', passwordVariable: 'GP')]) {
                    sh '''
                    git config user.email "jenkins-bot@automation.com"
                    git config user.name "Jenkins CI"

                    git add manifests/deployment.yaml

                    git commit -m "ci: update nginx image to $BUILD_NUMBER [skip ci]" || echo "No changes to commit"

                    git push https://${GU}:${GP}@${REPO_URL} HEAD:main
                    '''
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
