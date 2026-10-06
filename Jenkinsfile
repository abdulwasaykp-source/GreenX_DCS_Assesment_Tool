pipeline {

    agent any

    environment {
        DOCKER_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Code Clone') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/abdulwasaykp-source/GreenX_DCS_Assesment_Tool.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {

                    def scannerHome = tool name: 'SonarQube Scanner',
                        type: 'hudson.plugins.sonar.SonarRunnerInstallation'

                    withSonarQubeEnv('SonarQube') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                                -Dsonar.projectKey=greenx-dcs \
                                -Dsonar.projectName="GreenX DCS Assessment Tool" \
                                -Dsonar.sources=.
                        """
                    }
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                    docker compose build \
                        --build-arg DOCKER_TAG=${BUILD_NUMBER}
                '''
            }
        }

        stage('Application Deploy') {
            steps {
                sh '''
                    docker compose down || true
                    docker compose up -d
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

                        docker push abdulwasaykp/greenx-backend:${BUILD_NUMBER}
                        docker push abdulwasaykp/greenx-frontend:${BUILD_NUMBER}

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Ubuntu') {
            steps {
                sshagent(['ubuntu-deploy-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no \
                            osboxes@192.168.18.179 \
                            "mkdir -p /home/osboxes/greenx-deployment"

                        scp -o StrictHostKeyChecking=no \
                            compose.deploy.yml \
                            osboxes@192.168.18.179:/home/osboxes/greenx-deployment/compose.deploy.yml

                        ssh -o StrictHostKeyChecking=no \
                            osboxes@192.168.18.179 "
                                cd /home/osboxes/greenx-deployment &&
                                export DOCKER_TAG=${BUILD_NUMBER} &&
                                docker volume create greenx_dcs_assesment_tool_mysql_data || true &&
                                docker compose -f compose.deploy.yml pull &&
                                docker compose -f compose.deploy.yml up -d
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
