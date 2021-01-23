pipeline {
    agent any
    triggers {
        /* Vamos a pullear el repo activamente porque en nuestra instalación local sería complicado usar Webhooks */
        pollSCM('* * * * */1')
    }

    options {
        disableConcurrentBuilds()
    }
    environment {
        registry = "escarti/geekshub-django" /* Usad vuestro docker-hub registry */
        registryCredential = 'Docker'
        imageTag = "${env.GIT_BRANCH + '_' + env.BUILD_NUMBER}"
        apiServer = "https://192.168.99.101:8443"
        devNamespace = "default"
        minikubeCredential = 'minikube-auth-token'

    }
    stages {   
        stage('Build image') {
            steps {
                script {
                    dockerImage = docker.build(registry + ":$imageTag", "--cache-from $registry:latest --network host .")
                }
            }
        }
        stage('Test') {
            steps {
                sh "IMAGE=$registry TAG=$imageTag docker-compose -f docker-compose_test.yaml up --abort-on-container-exit --exit-code-from webapp"
            }
        }     
        stage('Upload image to registry') {
            steps{
                script {
                    docker.withRegistry( 'https://registry.hub.docker.com', registryCredential ) {
                        dockerImage.push()
                        dockerImage.push('latest')
                    }
                }
            }
        }
        stage('Deploy to K8s') {
            steps{
                withKubeConfig([credentialsId: minikubeCredential,
                                serverUrl: apiServer,
                                namespace: devNamespace
                               ]) {
                    sh 'kubectl set image deployment/django django="$registry:$imageTag" --record'
                }
            }
        }
    }
}