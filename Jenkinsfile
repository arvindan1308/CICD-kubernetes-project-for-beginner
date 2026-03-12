pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        DOCKER_USER = "arvindan1308n"
        IMAGE_NAME = "nginx-prod"
        REPO_URL = "https://github.com/arvindan1308/CICD-kubernetes-project-for-beginner.git"
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

                    docker build -t $DOCKER_USER/$IMAGE_NAME:$BUILD_NUMBER \
                    -f apps/frontend/Dockerfile apps/frontend

                    docker push $DOCKER_USER/$IMAGE_NAME:$BUILD_NUMBER
                    '''
                }
            }
        }

        stage('Update Manifest') {
            steps {
                sh '''
                sed -i "s|$DOCKER_USER/$IMAGE_NAME:.*|$DOCKER_USER/$IMAGE_NAME:$BUILD_NUMBER|" manifests/deployment.yaml
                '''
            }
        }

            stage('Push Changes to Git') {
    steps {
        withCredentials([usernamePassword(credentialsId: 'git-creds', usernameVariable: 'GU', passwordVariable: 'GP')]) {
            sh '''
            git config user.email "jenkins@devops.com"
            git config user.name "jenkins-bot"

            git pull origin main

            git add manifests/deployment.yaml
            git commit -m "Update image to build $BUILD_NUMBER [skip ci]" || echo "No changes"

            git push https://$GU:$GP@github.com/arvindan1308/CICD-kubernetes-project-for-beginner.git HEAD:main
            '''
        }
    }
}
    }
}