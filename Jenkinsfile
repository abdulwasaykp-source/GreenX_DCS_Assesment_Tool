pipeline {

    agent any

    environment {
        BACKEND_IMAGE  = "abdulwasaykp/greenx-backend"
        FRONTEND_IMAGE = "abdulwasaykp/greenx-frontend"
        DOCKER_TAG     = "${BUILD_NUMBER}"
    }

    stages {

        stage('Code Clone') {
            steps {
                git branch: 'main', url: 'https://github.com/abdulwasaykp-source/GreenX_DCS_Assesment_Tool.git'
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t ${BACKEND_IMAGE}:${DOCKER_TAG} -t ${BACKEND_IMAGE}:latest GreenX_DCS_Assesment_Tool_Backend
                    docker build -t ${FRONTEND_IMAGE}:${DOCKER_TAG} -t ${FRONTEND_IMAGE}:latest greenX-assessment-tool-frontend
                '''
            }
        }

        stage('Docker Hub Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhubcred',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                            -u "$DOCKER_USERNAME" \
                            --password-stdin

                        docker push ${BACKEND_IMAGE}:${DOCKER_TAG}
                        docker push ${BACKEND_IMAGE}:latest
                        docker push ${FRONTEND_IMAGE}:${DOCKER_TAG}
                        docker push ${FRONTEND_IMAGE}:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Ubuntu') {
            steps {
                sshagent(['ubuntu-deploy-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no osboxes@192.168.18.179 "
                            docker network create greenx-net || true &&
                            docker pull ${BACKEND_IMAGE}:${DOCKER_TAG} &&
                            docker pull ${FRONTEND_IMAGE}:${DOCKER_TAG} &&
                            docker rm -f greenx-backend greenx-frontend || true &&
                            docker run -d \
                                --name greenx-backend \
                                --network greenx-net \
                                -p 8000:8000 \
                                ${BACKEND_IMAGE}:${DOCKER_TAG} &&
                            docker run -d \
                                --name greenx-frontend \
                                --network greenx-net \
                                -p 3000:80 \
                                ${FRONTEND_IMAGE}:${DOCKER_TAG}
                        "

                        echo "Application deployed successfully."
                        echo "Frontend URL: http://192.168.18.179:3000"
                        echo "Backend URL:  http://192.168.18.179:8000"
                    '''
                }
            }
        }
    }
}
