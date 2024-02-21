pipeline {
    agent any

    environment {
        NODEJS_HOME = tool name: 'NodeJS', type: 'jenkins.plugins.nodejs.tools.NodeJSInstallation'
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    checkout scm
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    sh "${NODEJS_HOME}/bin/npm install"
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    sh "${NODEJS_HOME}/bin/npm test"
                }
            }
        }

        stage('Build Docker Image') {
            when {
                anyOf {
                    branch 'main'
                    branch 'dev'
                }
            }
            steps {
                script {
                    def dockerTag = ""
                    if (env.BRANCH_NAME == 'main') {
                        dockerTag = 'nodemain:v1.0'
                    } else if (env.BRANCH_NAME == 'dev') {
                        dockerTag = 'nodedev:v1.0'
                    }
                    sh "docker build -t ${dockerTag} ."
                }
            }
        }

        stage('Stop and Remove Containers') {
            steps {
                script {
                    sh 'docker stop $(docker ps -aq)'
                    sh 'docker rm $(docker ps -aq)'
                }
            }
        }

        stage('Run Docker Container') {
            when {
                anyOf {
                    branch 'main'
                    branch 'dev'
                }
            }
            steps {
                script {
                    def port = ""
                    if (env.BRANCH_NAME == 'main') {
                        port = '3000'
                    } else if (env.BRANCH_NAME == 'dev') {
                        port = '3001'
                    }
                    sh "docker run -d --expose ${port} -p ${port}:3000 ${dockerTag}"
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful'
        }
        failure {
            echo 'Deployment failed'
        }
    }
}