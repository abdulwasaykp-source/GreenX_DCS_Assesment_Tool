pipeline {

    agent any

    environment {
        DOCKER_TAG = "${BUILD_NUMBER}"

        SONAR_PROJECT = "greenx-dcs"

        FRONTEND_URL = "http://192.168.18.179:3000"
        BACKEND_URL  = "http://192.168.18.179:8000"

        DEVELOPER_EMAIL = "abdulwasaykp@gmail.com"
        CLIENT_EMAIL    = "kazamch749@gmail.com"
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
                    def scanner = tool(
                        name: 'SonarQube Scanner',
                        type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                    )

                    withSonarQubeEnv('SonarQube') {
                        sh """
                            ${scanner}/bin/sonar-scanner \
                            -Dsonar.projectKey=${SONAR_PROJECT} \
                            -Dsonar.projectName="GreenX DCS Assessment Tool" \
                            -Dsonar.sources=.
                        """
                    }
                }
            }
        }

        stage('SonarQube Quality Gate') {
            steps {
                script {
                    timeout(time: 5, unit: 'MINUTES') {
                        def result = waitForQualityGate(abortPipeline: false)
                        env.SONAR_STATUS = result.status
                    }
                }
            }
        }

        stage('SonarQube Email') {
            steps {
                script {
                    emailext(
                        to: "${DEVELOPER_EMAIL}",
                        subject: "SonarQube Report - Build #${BUILD_NUMBER}",
                        mimeType: 'text/html',
                        body: """
                            <h2>GreenX DCS - SonarQube Report</h2>

                            <p><b>Build:</b> #${BUILD_NUMBER}</p>
                            <p><b>Quality Gate:</b> ${SONAR_STATUS}</p>

                            <h3>SonarQube Dashboard</h3>

                            <a href="http://192.168.18.97:9000/dashboard?id=${SONAR_PROJECT}">
                                Open SonarQube Report
                            </a>

                            <hr>

                            <p>
                                SonarQube analysis complete hai.
                                Jenkins mein <b>Accept / Next</b> press
                                karke pipeline continue karo.
                            </p>
                        """
                    )
                }
            }
        }

        stage('Manual Approval') {
            steps {
                timeout(time: 30, unit: 'MINUTES') {
                    input(
                        message: "SonarQube: ${SONAR_STATUS}\n\nContinue pipeline?",
                        ok: "Accept / Next"
                    )
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

        stage('Docker Hub Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhubcred',
                        usernameVariable: 'USER',
                        passwordVariable: 'PASS'
                    )
                ]) {
                    sh '''
                        echo "$PASS" | docker login \
                            -u "$USER" --password-stdin

                        docker push abdulwasaykp/greenx-backend:${BUILD_NUMBER}
                        docker push abdulwasaykp/greenx-frontend:${BUILD_NUMBER}

                        docker logout
                    '''
                }
            }
        }

        stage('Local Deploy') {
            steps {
                sh '''
                    docker compose down || true
                    docker compose up -d
                '''
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
                            osboxes@192.168.18.179:/home/osboxes/greenx-deployment/

                        ssh -o StrictHostKeyChecking=no \
                            osboxes@192.168.18.179 "
                            cd /home/osboxes/greenx-deployment &&
                            export DOCKER_TAG=${BUILD_NUMBER} &&
                            docker volume create greenx_dcs_assesment_tool_mysql_data || true &&
                            docker compose -f compose.deploy.yml pull &&
                            docker compose -f compose.deploy.yml up -d
                            "
                    '''
                }
            }
        }

        stage('Client Email') {
            steps {
                emailext(
                    to: "${CLIENT_EMAIL}",
                    subject: "GreenX DCS Deployed - Build #${BUILD_NUMBER}",
                    mimeType: 'text/html',
                    body: """
                        <h2>GreenX DCS Deployment Successful</h2>

                        <p><b>Build:</b> #${BUILD_NUMBER}</p>

                        <h3>Application URLs</h3>

                        <p>
                            <b>Frontend:</b>
                            <a href="${FRONTEND_URL}">
                                ${FRONTEND_URL}
                            </a>
                        </p>

                        <p>
                            <b>Backend:</b>
                            <a href="${BACKEND_URL}">
                                ${BACKEND_URL}
                            </a>
                        </p>

                        <p>
                            You can now access and test the application
                            using the above URLs.
                        </p>
                    """
                )
            }
        }
    }

    post {
        failure {
            emailext(
                to: "${DEVELOPER_EMAIL}",
                subject: "GreenX Pipeline Failed - Build #${BUILD_NUMBER}",
                body: "Pipeline failed. Please check Jenkins Console Output."
            )
        }

        success {
            echo "GreenX deployment completed successfully."
        }
    }
}
