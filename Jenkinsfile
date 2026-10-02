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

        stage('Application Deploy') {
            steps {
                sh '''
                    docker compose down
                    docker compose pull
                    docker compose up -d
                '''
            }
        }
    }
}


