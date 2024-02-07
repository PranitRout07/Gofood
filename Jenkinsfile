pipeline {

    agent any
    
    stages {

        stage('Build Docker Images'){
            steps{
                
                bat 'docker build -t gofood-frontend .'
                bat 'docker build -t gofood-backend ./backend'
                
            }

        }
        stage('Push Docker Dmage To Dockerhub'){
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dockerPass', usernameVariable: 'dockerUser')]) {
                   bat "docker login -u ${env.dockerUser} -p ${env.dockerPass}"
                   bat "docker tag gofood-frontend ${env.dockerUser}/gofood-frontend:latest"
                   bat "docker push ${env.dockerUser}/gofood-frontend:latest "
                   bat "docker tag gofood-backend ${env.dockerUser}/gofood-backend:latest"
                   bat "docker push ${env.dockerUser}/gofood-backend:latest "
                }
            }

        }
        stage('Scan the docker image of frontend with trivy') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dockerPass', usernameVariable: 'dockerUser')]) {
            
                    bat "docker run --rm -v D:/trivy-report/:/root/.cache/ aquasec/trivy:0.18.3 image ${env.dockerUser}/gofood-frontend:latest"
                    
                }
            }

        }
        stage('Scan the docker image of backend with trivy') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dockerPass', usernameVariable: 'dockerUser')]) {
            
                    bat "docker run --rm -v D:/trivy-report/:/root/.cache/ aquasec/trivy:0.18.3 image ${env.dockerUser}/gofood-backend:latest"
                    
                }
            }

        }
        stage('Deploy'){
            steps{
                bat 'docker-compose up -d'
            }
        }
        

    }
        post {
        always {
            script {
                def textColor
                switch (currentBuild.currentResult) {
                    case 'SUCCESS':
                        textColor = 'green';
                        break
                    default:
                        textColor = 'red';
                }

                emailext (
                    subject: "Pipeline Status: ${currentBuild.currentResult}",
                    body: """<html>
                                <body>
                                    <p><b>Project:</b> <i>${JOB_NAME}</i></p>
                                    <p><b>Build Number:</b> <i>${BUILD_NUMBER}</i></p>
                                    <p><b>Build URL:</b> ${BUILD_URL}</p>
                                    <p style='color: ${textColor};'>Build Result: ${currentBuild.currentResult}</p>
                                </body>
                            </html>""",
                    to: 'pranitrout72@gmail.com',
                    from: 'jenkins@example.com',
                    replyTo: 'jenkins@example.com',
                    mimeType: 'text/html'
                )
            }
        }
    }
}
