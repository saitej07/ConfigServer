pipeline {
    agent any

    environment {
        MVN_HOME = tool name: 'maven', type: 'maven'
        MVN_CMD = "${MVN_HOME}/bin/mvn"
        APP_NAME = "configserver"
        DOCKER_IMAGE = "saitej150/${APP_NAME}:${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    extensions: [],
                    userRemoteConfigs: [[url: 'https://github.com/saitej07/ConfigServer.git']]])
            }
        }

        stage('Build JAR') {
            steps {
                bat "${env.MVN_CMD} clean package -DskipTests"
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    bat "echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t ${env.DOCKER_IMAGE} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                bat "docker push ${env.DOCKER_IMAGE}"
            }
        }

        stage('Deploy') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-creds', variable: 'KUBECONFIG_FILE')]) {
                    bat """
                        set KUBECONFIG=%KUBECONFIG_FILE%
                        kubectl apply -f k8s/deployment.yaml
                        kubectl rollout status deployment/${env.APP_NAME}-deployment
                    """
                }
            }
        }
    }
}
