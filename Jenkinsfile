node {
    def mvnHome = tool name: 'maven', type: 'maven'
    def mvnCMD = "${mvnHome}/bin/mvn"
    def appName = "configserver"
    def dockerImage = "saitej150/${appName}:${BUILD_NUMBER}"

    stage('Checkout') {
        checkout([$class: 'GitSCM',
            branches: [[name: '*/main']],
            extensions: [],
            userRemoteConfigs: [[url: 'https://github.com/saitej07/ConfigServer.git']]])
    }

    stage('Build JAR') {
        sh "${mvnCMD} clean package -DskipTests"
    }

    stage('Build Docker Image') {
        sh "docker build -t ${dockerImage} ."
    }

    stage('Push Docker Image') {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
            sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
            sh "docker push ${dockerImage}"
        }
    }

    stage('Deploy') {
    withCredentials([file(credentialsId: 'kubeconfig-creds', variable: 'KUBECONFIG_FILE')]) {
        sh """
           export KUBECONFIG=$KUBECONFIG_FILE
           kubectl apply -f k8s/deployment.yaml
           kubectl rollout status deployment/${appName}-deployment
        """
    }
}
}
