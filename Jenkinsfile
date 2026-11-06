def run(cmd) {
    if (isUnix()) {
        sh cmd
    } else {
        bat cmd
    }
}

pipeline {
    agent any

    environment {
        REGISTRY = 'hellojenk123.azurecr.io'
        IMAGE    = 'hello-jenkins'
    }

    stages {

        stage('Build') {
            steps {
                script {
                    run "docker build -t ${IMAGE}:${BUILD_NUMBER} ."
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    run "docker run --rm ${IMAGE}:${BUILD_NUMBER} nginx -t"
                }
            }
        }

        stage('Push') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'acr-creds',
                        usernameVariable: 'U',
                        passwordVariable: 'P'
                    )]) {

                        if (isUnix()) {
                            sh 'printf "%s" "$P" | docker login "$REGISTRY" -u "$U" --password-stdin'
                        } else {
                            bat 'echo %P% | docker login %REGISTRY% -u %U% --password-stdin'
                        }
                    }

                    run "docker tag ${IMAGE}:${BUILD_NUMBER} ${REGISTRY}/${IMAGE}:latest"

                    run "docker push ${REGISTRY}/${IMAGE}:latest"
                }
            }
        }
    }
}
