pipeline {
    agent {
        node {
            label 'manager'
        }
    }
    environment{
        scannerHome = tool 'mysonar';
    }
    stages {
        stage('code') {
            steps {
                git "https://github.com/mmunna431/python-code-library-app.git"
            }
        }
        stage('cqa'){
            steps{
                withSonarQubeEnv('mysonar') {
                  sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=project1"
                }
            }
        }
        stage('image'){
            steps{
                sh '''
                    docker build -t dbimage database
                    docker build -t authimage auth
                    docker build -t bookimage book
                    docker build -t borrowimage borrow
                    docker build -t appimage .
                '''
            }
        }
        stage('scan'){
            steps{
                sh '''
                    trivy image dbimage
                    trivy image authimage
                    trivy image bookimage
                    trivy image borrowimage
                    trivy image appimage
                '''
            }
        }
        stage('tag'){
            steps{
                sh '''
                    docker tag dbimage mmunna431/py-app:dbimage
                    docker tag authimage mmunna431/py-app:authimage
                    docker tag bookimage mmunna431/py-app:bookimage
                    docker tag borrowimage mmunna431/py-app:borrowimage
                    docker tag appimage mmunna431/py-app:appimage
                '''
            }
        }
        stage('Registry'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'dockerhubid') {
                        sh 'docker push mmunna431/py-app:dbimage'
                        sh 'docker push mmunna431/py-app:authimage'
                        sh 'docker push mmunna431/py-app:bookimage'
                        sh 'docker push mmunna431/py-app:borrowimage'
                        sh 'docker push mmunna431/py-app:appimage' 
                    }
                }
            }
        }
        stage('deploy'){
            steps{
                sh 'docker stack deploy pyappStack --compose-file=compose.yml'
            }
        }
    }
}
