pipeline {
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }
    agent any
    triggers {
        githubPush()
    }
    
    environment {
        DOCKER_USER = "arvindan1308n"
        IMAGE_NAME = "nginx-gitops"
        // Use the HTTPS URL for the push logic
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
                // Use double quotes for the sh command to allow Jenkins to inject the environment variables
                sh "sed -i 's|${DOCKER_USER}/${IMAGE_NAME}:.*|${DOCKER_USER}/${IMAGE_NAME}:${env.BUILD_NUMBER}|' manifests/deployment.yaml"
            }
        }

        stage('Push Changes to Git') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-creds', usernameVariable: 'GU', passwordVariable: 'GP')]) {
                    sh '''
                    git config user.email "jenkins-bot@automation.com"
                    git config user.name "Jenkins CI"

                    git add manifests/deployment.yaml
                    
                    # Ensure BUILD_NUMBER is inside the string to be captured in the commit
                    # git commit -m "Auto-update image to tag ${BUILD_NUMBER}"
                    git commit -m "Update image to build $BUILD_NUMBER [skip ci]" || echo "No changes are made"


                    # The key fix: push current HEAD to remote main using credentials
                    git push https://${GU}:${GP}@${REPO_URL} HEAD:main
                    '''
                }
            }
        }
    }

    post {
        always {
            // Optional: Clean up workspace to keep the agent healthy
            cleanWs()
        }
    }
}
