pipeline {
    agent any
    environment {
        PATH = "/usr/local/bin:$PATH"
        DOCKER_HOST = "unix:///var/run/docker.sock"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                sh 'docker run --rm -u root -v $(pwd):/opt -w /opt node:14 npm install'
            }
        }
        stage('Test') {
            steps {
                sh 'docker run --rm -u root -v $(pwd):/opt -w /opt -e CI=true node:14 npm test'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                         sh 'docker build -t nodemain:v1.0 .'
                    } else if (env.BRANCH_NAME == 'dev') {
                         sh 'docker build -t nodedev:v1.0 .'
                    } else {
                        echo "Skipping docker build for branch ${env.BRANCH_NAME}"
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        // Stop existing container if running
                        sh 'docker rm -f app-main || true'
                        sh 'docker run -d --expose 3000 -p 3000:3000 --name app-main nodemain:v1.0'
                    } else if (env.BRANCH_NAME == 'dev') {
                        // Stop existing container if running
                        sh 'docker rm -f app-dev || true'
                        sh 'docker run -d --expose 3001 -p 3001:3000 --name app-dev nodedev:v1.0'
                    }
                }
            }
        }
    }
}
