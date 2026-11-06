
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
        REGISTRY  = 'webazure.azurecr.io'
        IMAGE     = 'hello-azure'
        IMAGE_TAG = 'latest'
    }

    stages {

        // Stage 1: Verify source files from GitHub
        stage('Verify Source Files') {
            steps {
                script {
                    if (isUnix()) {
                        sh '''
                            echo "========== GIT COMMIT =========="
                            git log -1 --oneline

                            echo "========== FILE LIST =========="
                            ls -lah

                            echo "========== HTML FILE SIZE =========="
                            wc -c index.html

                            echo "========== HTML CONTENT =========="
                            cat index.html

                            echo "========== HTML CHECKSUM =========="
                            sha256sum index.html
                        '''
                    } else {
                        bat '''
                            echo ========== GIT COMMIT ==========
                            git log -1 --oneline

                            echo ========== FILE LIST ==========
                            dir

                            echo ========== HTML CONTENT ==========
                            type index.html
                        '''
                    }
                }
            }
        }

        // Stage 2: Build Docker image
        stage('Build') {
            steps {
                script {
                    run "docker build --no-cache --pull -t ${IMAGE}:${IMAGE_TAG} ."
                }
            }
        }

        // Stage 3: Test Nginx configuration and HTML
        stage('Test') {
            steps {
                script {
                    run "docker run --rm ${IMAGE}:${IMAGE_TAG} nginx -t"

                    if (isUnix()) {
                        sh """
                            echo "========== VERIFY HTML IN DOCKER IMAGE =========="

                            echo "Source HTML:"
                            sha256sum index.html

                            echo "Container HTML:"
                            docker run --rm --entrypoint sha256sum ${IMAGE}:${IMAGE_TAG} /usr/share/nginx/html/index.html
                        """
                    }
                }
            }
        }

        // Stage 4: Login and push image to Azure Container Registry
        stage('Push') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'acr-cred',
                        usernameVariable: 'U',
                        passwordVariable: 'P'
                    )]) {

                        if (isUnix()) {
                            sh '''
                                printf "%s" "$P" | docker login "$REGISTRY" -u "$U" --password-stdin
                            '''
                        } else {
                            bat '''
                                echo %P% | docker login %REGISTRY% -u %U% --password-stdin
                            '''
                        }
                    }

                    run "docker tag ${IMAGE}:${IMAGE_TAG} ${REGISTRY}/${IMAGE}:${IMAGE_TAG}"

                    run "docker push ${REGISTRY}/${IMAGE}:${IMAGE_TAG}"
                }
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Docker image built, tested and pushed to Azure Container Registry.'
            echo 'Check Azure Web App for the updated container.'
        }

        failure {
            echo 'FAILED: Check the Jenkins console output for the failed stage.'
        }
    }
}
