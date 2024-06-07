pipeline {
    agent {
        node {
            label 'slave'
        }
    }
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
                echo 'Cleaning up completed'
            }
        }

        stage('Check for backend package.json') {
            steps {
                script {
                    echo 'Checking if package.json exists in the expected directory...'
                    dir('api') {
                        sh 'ls -l package.json'
                    }
                }
            }
        }

        stage('Build and Push Backend Docker Images') {
            steps {
                echo "Now we build backend images and push to Docker Hub"
                dir('api') {
                    script {
                        def packageJSON = readJSON file: 'package.json'
                        def packageJSONVersion = packageJSON.version
                        withDockerRegistry([credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/']) {
                            sh """
                                docker build -t secretrulerkings/backend-app:${packageJSONVersion} .
                                docker push secretrulerkings/backend-app:${packageJSONVersion}
                            """
                        }
                    }
                }
            }
        }

        stage('Check for frontend package.json') {
            steps {
                script {
                    echo 'Checking if package.json exists in the frontend directory...'
                    dir('webapp') {
                        sh 'ls -l package.json'
                    }
                }
            }
        }

        stage('Build and Push Frontend Docker Images') {
            steps {
                echo "Now we build frontend images and push to Docker Hub"
                dir('webapp') {
                    script {
                        def packageJSON = readJSON file: 'package.json'
                        def packageJSONVersion = packageJSON.version
                        withDockerRegistry([credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/']) {
                            sh """
                                docker build -t secretrulerkings/frontend-app:${packageJSONVersion} .
                                docker push secretrulerkings/frontend-app:${packageJSONVersion}
                            """
                        }
                    }
                }
            }
        }

        stage('Configure EKS Cluster') {
            steps {
                sh 'aws eks --region eu-west-1 update-kubeconfig --name lms-cluster'
            }
        }

        stage('DB Deployment') {
            steps {
                sh 'cd api && kubectl apply -f postgres-deployment.yaml'
            }
        }

        stage('DB Cluster IP Service') {
            steps {
                sh 'cd api && kubectl apply -f postgres-cluster-ip-service.yaml'
            }
        }

        stage('DB Persistent Volume') {
            steps {
                sh 'cd api && kubectl apply -f database-persistent-volume.yaml'
            }
        }

        stage('Backend Deployment') {
            steps {
                dir('api') {
                    script {
                        def packageJSON = readJSON file: 'package.json'
                        def packageJSONVersion = packageJSON.version
                        sh """
                            sed -i 's#image: secretrulerkings/backend-app:.*#image: secretrulerkings/backend-app:${packageJSONVersion}#' api-deployment.yaml
                            kubectl apply -f api-deployment.yaml
                        """
                    }
                }
            }
        }

        stage('Backend Load Balancer Service') {
            steps {
                sh 'cd api && kubectl apply -f api-load-balancer-service.yaml'
            }
        }

        stage('Frontend Deployment') {
            steps {
                dir('webapp') {
                    script {
                        def packageJSON = readJSON file: 'package.json'
                        def packageJSONVersion = packageJSON.version
                        sh """
                            sed -i 's#image: secretrulerkings/frontend-app:.*#image: secretrulerkings/frontend-app:${packageJSONVersion}#' frontend-deployment.yaml
                            kubectl apply -f frontend-deployment.yaml
                        """
                    }
                }
            }
        }

        stage('Frontend Load Balancer Service') {
            steps {
                sh 'cd webapp && kubectl apply -f frontend-load-balancer-service.yaml'
            }
        }
    }
}
