pipeline {
    agent any
    environment {
        registry = "escarti/geekshub-django"
        registryCredential = 'docker-registry'
    }
    stages {
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }
        stage('Build image') {
            steps {
                script {
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
            }
        }
        stage('Deploy Image') {
            steps{
                script {
                    docker.withRegistry( '', registryCredential ) {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Deploy to K8s') {
            steps {
                echo 'Deploying....'
            }
        }
    }
    node {
        stage('List pods') {
            withKubeConfig([credentialsId: 'minikube-auth-token',
                            serverUrl: '192.168.99.101',
                            namespace: 'default'
                            ]) {
              sh 'kubectl get pods'
            }
        }
    }

}