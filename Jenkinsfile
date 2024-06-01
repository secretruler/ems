pipeline {
    agent any
    stages {

        stage('Cleaning up docker') {
            steps {
                echo "Cleaning up docker"
                sh 'docker rm $(docker ps -a -q) && docker rmi -f $(docker images -a -q) && docker system prune -f'
                echo 'cleaning up completed'
            }
        }

        stage('Runing a Database container') {
            steps {
                echo "building a database container"
                sh 'docker container run -dt --name lms-db -e POSTGRES_PASSWORD=password postgres'
                echo 'building database container completed'
            }
        }
        stage('Build and Push Docker Images') {
            steps {
                echo "Now we build images and push to Docker Hub"
                dir('lms/api') {
                    script {
                        def packageJSON = readJSON file: 'package.json'
                        def packageJSONVersion = packageJSON.version
                        withDockerRegistry(credentialsId: 'dockerhub-cred') {
                            sh '''
                                rm -f .env
                                echo "MODE=production" > .env
                                echo "PORT=8080" >> .env
                                echo "DATABASE_URL=postgresql://postgres:password@lms-db:5432/postgres" >> .env
                                docker build -t secretrulerkings/backend-app:${packageJSONVersion} .
                                docker push secretrulerkings/backend-app:${packageJSONVersion}
                            '''
                        }
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                echo "deploy stage"
            }
        }
    }
}
