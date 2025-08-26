// Global variables for the pipeline, accessible to all stages.
def appPort
def appName
def appTag
def dockerRepo = "your-dockerhub-username" // <-- CHANGE THIS TO YOUR DOCKER HUB USERNAME

pipeline {
    agent any

    stages {
        stage('Initialize Environment') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        appName = "nodemain"
                        appPort = 3000
                    } else if (env.BRANCH_NAME == 'dev') {
                        appName = "nodedev"
                        appPort = 3001
                    }
                    appTag = "${appName}:v1.0"
                    echo "Initializing environment for branch '${env.BRANCH_NAME}'."
                    echo "App Name: ${appName}"
                    echo "App Port: ${appPort}"
                    echo "App Tag: ${appTag}"
                }
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                // Build the Docker image with the dynamically set tag.
                sh "docker build -t ${appTag} ."
                // Push the image to your Docker Hub repository.
                // Using withCredentials for secure access to Docker Hub credentials.
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                    sh "docker tag ${appTag} ${dockerRepo}/${appTag}"
                    sh "docker push ${dockerRepo}/${appTag}"
                }
            }
        }

        stage('Deploy') {
            steps {
                // Stop and remove the old container for the current branch.
                sh "docker container stop \$(docker container ls -q --filter ancestor=${appTag}) || true"
                sh "docker container rm \$(docker container ls -q -a --filter ancestor=${appTag}) || true"
                // Run the new container, exposing the correct port.
                sh "docker run -d -p ${appPort}:3000 --name ${appName} ${appTag}"
            }
        }
    }
}