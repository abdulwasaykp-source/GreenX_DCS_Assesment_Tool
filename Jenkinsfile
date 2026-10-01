```groovy
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
                git branch: 'main',
                    url: 'https://github.com/abdulwasaykp-source/GreenX_DCS_Assesment_Tool.git'
            }
        }


        stage('Docker Build') {
            steps {
                sh '''
                    docker build \
                        -t ${BACKEND_IMAGE}:${DOCKER_TAG} \
                        GreenX_DCS_Assesment_Tool_Backend

                    docker build \
                        -t ${FRONTEND_IMAGE}:${DOCKER_TAG} \
                        greenX-assessment-tool-frontend
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
                        docker push ${FRONTEND_IMAGE}:${DOCKER_TAG}

                        docker logout
                    '''
                }
            }
        }


        stage('Deploy to Server 2') {

            environment {
                DEPLOY_HOST = "192.168.18.179"
                DEPLOY_USER = "osboxes"
                DEPLOY_DIR  = "/home/osboxes/greenx-deployment"
            }

            steps {

                sshagent(['ubuntu-deploy-key']) {

                    sh '''
                        echo "========================================"
                        echo " Deploying GreenX to Server 2"
                        echo " Server: ${DEPLOY_HOST}"
                        echo " User:   ${DEPLOY_USER}"
                        echo " Tag:    ${DOCKER_TAG}"
                        echo "========================================"


                        echo "Creating deployment directory..."

                        ssh -o StrictHostKeyChecking=no \
                            ${DEPLOY_USER}@${DEPLOY_HOST} \
                            "mkdir -p ${DEPLOY_DIR}"


                        echo "Copying docker-compose.yml..."

                        scp -o StrictHostKeyChecking=no \
                            docker-compose.yml \
                            ${DEPLOY_USER}@${DEPLOY_HOST}:${DEPLOY_DIR}/docker-compose.yml


                        echo "Copying backend .env..."

                        scp -o StrictHostKeyChecking=no \
                            GreenX_DCS_Assesment_Tool_Backend/.env \
                            ${DEPLOY_USER}@${DEPLOY_HOST}:${DEPLOY_DIR}/.env


                        echo "Deploying application..."

                        ssh -o StrictHostKeyChecking=no \
                            ${DEPLOY_USER}@${DEPLOY_HOST} "
                            
                                cd ${DEPLOY_DIR}

                                export DOCKER_TAG=${DOCKER_TAG}

                                echo '========================================'
                                echo ' Pulling Backend Image'
                                echo '========================================'

                                docker pull ${BACKEND_IMAGE}:${DOCKER_TAG}


                                echo '========================================'
                                echo ' Pulling Frontend Image'
                                echo '========================================'

                                docker pull ${FRONTEND_IMAGE}:${DOCKER_TAG}


                                echo '========================================'
                                echo ' Starting GreenX'
                                echo '========================================'

                                docker compose up -d --remove-orphans


                                echo '========================================'
                                echo ' Deployment Status'
                                echo '========================================'

                                docker compose ps

                            "


                        echo "========================================"
                        echo " Deployment completed"
                        echo "========================================"
                    '''
                }
            }
        }
    }
}
```

