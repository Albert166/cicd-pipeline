def appPort
def appName
def appTag

pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        appName = "nodemain"
                        appPort = 3000
                    } else if (env.BRANCH_NAME == 'dev') {
                        appName = "nodedev"
                        appPort = 3001
                    }
                    appTag = appName + ":" + "v1.0"
                }
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                sh './scripts/build.sh'
            
            }
        }

        stage('Test') {
            steps {
                sh './scripts/test.sh'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${appTag} .'
            }
        }

        stage('Deploy') {
            steps {
                sh 'docker container stop $(docker container ls -q --filter ancestor=${appTag}) || true'
                sh 'docker container rm $(docker container ls -q -a --filter ancestor=${appTag}) || true'
                sh 'docker run -d -p ${appPort}:3000 --name ${appName} ${appTag}'
            }
        }
    }
}