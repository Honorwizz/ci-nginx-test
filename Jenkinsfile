pipeline {
    agent any

    environment {
        IMAGE_NAME = "nginx:latest"
        CONTAINER_NAME = "ci_nginx_test"
        HOST_PORT = "9889"
        CONTAINER_PORT = "80"
        TELEGRAM_BOT_TOKEN = "7397473657:AAGHn2gysmSRPEzebRIdca3ZBe-hxpD6Th4"
        TELEGRAM_CHAT_ID = "1242338965"
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/Honorwizz/ci-nginx-test.git']]
                ])
            }
        }

        stage('Build and Run Nginx Container') {
            steps {
                script {
                    sh """
                        docker run -d --rm --name ${CONTAINER_NAME} \
                        -p ${HOST_PORT}:${CONTAINER_PORT} \
                        -v `pwd`/index.html:/usr/share/nginx/html/index.html \
                        ${IMAGE_NAME}
                    """
                }
            }
        }

        stage('Verify HTTP Response') {
            steps {
                script {
                    def response = sh(script: "curl -o /dev/null -s -w '%{http_code}' http://localhost:${HOST_PORT}", returnStdout: true).trim()
                    if (response != "200") {
                        error "HTTP response code is not 200!"
                    }
                }
            }
        }

        stage('Verify MD5 Checksum') {
            steps {
                script {
                    def local_md5 = sh(script: "md5sum index.html | awk '{ print \$1 }'", returnStdout: true).trim()
                    def remote_md5 = sh(script: "curl -s http://localhost:${HOST_PORT} | md5sum | awk '{ print \$1 }'", returnStdout: true).trim()
                    if (local_md5 != remote_md5) {
                        error "MD5 checksum mismatch!"
                    }
                }
            }
        }

        stage('Cleanup') {
            steps {
                script {
                    sh "docker stop ${CONTAINER_NAME}"
                }
            }
        }
    }

    post {
        failure {
            script {
                def message = "❌ Jenkins Pipeline Failed\n" +
                              "🔹 Job: ${env.JOB_NAME}\n" +
                              "🔹 Build: ${env.BUILD_NUMBER}\n" +
                              "🔹 Reason: ${currentBuild.result}\n" +
                              "🔹 Logs: ${env.BUILD_URL}"

                sh """
                    curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage" \
                    -d chat_id=${TELEGRAM_CHAT_ID} \
                    -d text=\"${message}\" \
                    -d parse_mode="Markdown"
                """
            }
        }
    }
}
