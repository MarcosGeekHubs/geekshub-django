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
                sh "docker-compose -f docker-compose_test.yaml up -e IMAGE=$registry -e TAG=$imageTag --abort-on-container-exit --exit-code-from webapp"
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
    }
}