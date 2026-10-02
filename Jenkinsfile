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
                echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin

                docker push abdulwasaykp/greenx-backend:${BUILD_NUMBER}
                docker push abdulwasaykp/greenx-frontend:${BUILD_NUMBER}

                docker logout
            '''
           }
       }
   }
}


