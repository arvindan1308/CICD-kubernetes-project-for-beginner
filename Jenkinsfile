pipeline {
    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        DOCKER_USER = "arvindan1308n"
        IMAGE_NAME = "nginx-gitops"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Prevent Jenkins Loop') {
            steps {
                script {
                    def commitMessage = sh(
                        script: "git log -1 --pretty=%B",
                        returnStdout: true
                    ).trim()

                    if (commitMessage.contains("[skip ci]")) {
                        echo "Skipping build triggered by Jenkins commit"
                        currentBuild.result = 'ABORTED'
                        error("Stopping pipeline to prevent loop")
                    }
                }
            }
        }

        stage('Build & Push Docker') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'U', passwordVariable: 'P')]) {
                    sh '''
                    echo $P | docker login -u $U --password-stdin
                    docker build -t arvindan1308n/nginx-gitops:$BUILD_NUMBER -f apps/frontend/Dockerfile apps/frontend
                    docker push arvindan1308n/nginx-gitops:$BUILD_NUMBER
                    '''
                }
            }
        }

        stage('Update Manifest') {
            steps {
                sh """
                sed -i 's|arvindan1308n/nginx-gitops:.*|arvindan1308n/nginx-gitops:${BUILD_NUMBER}|' manifests/deployment.yaml
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

                    git commit -m "ci: update nginx image to $BUILD_NUMBER [skip ci] || echo "No changes

                    git push https://${GU}:${GP}@github.com/arvindan1308/CICD-kubernetes-project-for-beginner.git HEAD:main
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
