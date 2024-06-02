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
                echo 'Cleaning up completed'
            }
        }

        stage('Clean up Kubernetes Resources') {
            steps {
                echo "Cleaning up Kubernetes resources"
                script {
                    sh """
                        kubectl delete all --all -n akhilkings || true
                    """
                }
                echo 'Kubernetes clean up completed'
            }
        }
        
        stage('Check for backend package.json') {
            steps {
                script {
                    echo 'Checking if package.json exists in the api directory...'
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
                        withDockerRegistry([credentialsId: 'dockerhub-cred', url: 'https://index.docker.io/v1/']) {
                            sh """
                                docker build -t secretrulerkings/backend-app:${packageJSONVersion} .
                                docker push secretrulerkings/backend-app:${packageJSONVersion}
                            """
                        }

                    }
                }
            }
        }

        stage('Deploy Backend to Kubernetes') {
            steps {
                echo "Deploying backend to Kubernetes"
                dir('api') {
                    script {
                        def packageJSON = readJSON file: 'package.json'
                        def packageJSONVersion = packageJSON.version
                        sh """
                            sed -i 's/\${packageJSONVersion}/${packageJSONVersion}/g' be-deployment.yml
                            kubectl apply -f akhilkings-namespace.yml
                            kubectl apply -f pg-secret.yml -n akhilkings
                            kubectl apply -f pg-deployment.yml -n akhilkings
                            kubectl apply -f pg-service.yml -n akhilkings
                            kubectl apply -f be-configmap.yml -n akhilkings
                            kubectl apply -f be-deployment.yml -n akhilkings
                            kubectl apply -f be-service.yml -n akhilkings
                            kubectl get pods -n akhilkings
                            kubectl get services -n akhilkings 
                        """
                    }
                }
            }
        }
        stage('Check for frontend package.json') {
            steps {
                script {
                    echo 'Checking if package.json exists in the webapp directory...'
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
                        withDockerRegistry([credentialsId: 'dockerhub-cred', url: 'https://index.docker.io/v1/']) {
                            sh """
                                docker build -t secretrulerkings/frontend-app:${packageJSONVersion} .
                                docker push secretrulerkings/frontend-app:${packageJSONVersion}
                            """
                        }
                    }
                }
            }
        }
        stage('Deploy Frontend to Kubernetes') {
            steps {
                echo "Deploying frontend to Kubernetes"
                dir('webapp') {
                    script {
                        def packageJSON = readJSON file: 'package.json'
                        def packageJSONVersion = packageJSON.version
                        sh """
                            sed -i 's/\${packageJSONVersion}/${packageJSONVersion}/g' fe-deployment.yml
                            kubectl apply -f fe-deployment.yml -n akhilkings
                            kubectl apply -f fe-service.yml -n akhilkings
                            kubectl get pods -n akhilkings
                            kubectl get services -n akhilkings
                        """
                    }
                }
            }
        }
    }
}
