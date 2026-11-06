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
        REGISTRY = 'webazure.azurecr.io'
        IMAGE    = 'hello-azure'
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
                        credentialsId: 'acr-cred',
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
