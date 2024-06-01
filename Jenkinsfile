pipeline {
    agent any
    options {
        buildDiscarder(logRotator(numToKeepStr: '1')) // Keep only the latest build
    }
    stages {
        stage('Cleaning up docker') {
            steps {
                echo "Cleaning up docker"
                script {
                    // Stop and remove all running containers
                    sh 'docker stop $(docker ps -a -q) || true'
                    sh 'docker rm $(docker ps -a -q) || true'
                    // Remove all Docker images
                    sh 'docker rmi -f $(docker images -a -q) || true'
                    // Clean up any other Docker resources
                    sh 'docker system prune -f || true'
                }
                echo 'cleaning up completed'
            }
        }
        stage('Running a Database container') {
            steps {
                echo "building a database container"
                sh 'docker container run -dt --name lms-db -e POSTGRES_PASSWORD=password postgres'
                echo 'building database container completed'
            }
        }
        stage('Check for package.json') {
            steps {
                script {
                    echo "Checking if package.json exists in the expected directory..."
                    sh 'ls -lR' // List all files and directories to check the structure
                    sh 'cat lms/webapp/package.json' // Display the content of package.json to verify its existence
                }
            }
        }
        stage('Build and Push Docker Images') {
            steps {
                echo "Now we build images and push to Docker Hub"
                dir('lms/api') {
                    script {
                        def packageJSON = readJSON file: 'webapp/package.json'
                        def packageJSONVersion = packageJSON.version
                        withDockerRegistry([credentialsId: 'dockerhub-cred', url: 'https://index.docker.io/v1/']) {
                            sh """
                                rm -f .env
                                echo "MODE=production" > .env
                                echo "PORT=8080" >> .env
                                echo "DATABASE_URL=postgresql://postgres:password@lms-db:5432/postgres" >> .env
                                docker build -t secretrulerkings/backend-app:${packageJSONVersion} .
                                docker push secretrulerkings/backend-app:${packageJSONVersion}
                            """
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
