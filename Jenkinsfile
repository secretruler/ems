pipeline {
    agent any

    environment {
        KUBECONFIG = '/var/run/secrets/kubernetes.io/serviceaccount/.kube/config'
    }
    
    stages {
        stage('Integrate Remote k8s with Jenkins') {
            steps {
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'production', contextName: '', credentialsId: 'SECRET_TOKEN', namespace: 'default', serverUrl: 'https://71DE4B8751047A8016EFEC2435B92D4F.gr7.eu-north-1.eks.amazonaws.com']]) {
                    sh 'curl -LO "https://storage.googleapis.com/kubernetes-release/release/v1.20.5/bin/linux/amd64/kubectl"'  
                    sh 'chmod u+x ./kubectl'  
                    sh './kubectl get nodes'
                    sh './kubectl config get-contexts'
                    sh './kubectl config current-context'
                }
            }
        }
        
        stage('Cleaning up Docker') {
            steps {
                echo "Cleaning up Docker"
                script {
                    sh 'docker stop $(docker ps -a -q) || true'
                    sh 'docker rm $(docker ps -a -q) || true'
                    sh 'docker rmi -f $(docker images -a -q) || true'
                    sh 'docker system prune -f || true'
                }
                echo 'Cleaning up completed'
            }
        }

        stage('Clean up Kubernetes Resources') {
            steps {
                echo "Cleaning up Kubernetes resources"
                script {
                    withKubeCredentials(kubectlCredentials: [[credentialsId: 'SECRET_TOKEN', clusterName: 'production']]) {
                        sh 'kubectl delete pod --all --namespace=default || true'
                        sh 'kubectl delete service --all --namespace=default || true'
                        sh 'kubectl delete replicasets --all --namespace=default || true'
                        sh 'kubectl delete deployments --all --namespace=default || true'
                        sh 'kubectl delete configmaps --all --namespace=default || true'
                        sh 'kubectl delete secrets --all --namespace=default || true'
                    }
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
                        withKubeCredentials(kubectlCredentials: [[credentialsId: 'SECRET_TOKEN', clusterName: 'production']]) {
                            sh """
                                sed -i 's/\${packageJSONVersion}/${packageJSONVersion}/g' be-deployment.yml
                                kubectl apply -f pg-secret.yml
                                kubectl apply -f pg-deployment.yml
                                kubectl apply -f pg-service.yml
                                kubectl apply -f be-configmap.yml
                                kubectl apply -f be-deployment.yml
                                kubectl apply -f be-service.yml
                                kubectl get pods
                                kubectl get services
                            """
                        }
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
                        withKubeCredentials(kubectlCredentials: [[credentialsId: 'SECRET_TOKEN', clusterName: 'production']]) {
                            sh """
                                sed -i 's/\${packageJSONVersion}/${packageJSONVersion}/g' fe-deployment.yml
                                kubectl apply -f fe-deployment.yml
                                kubectl apply -f fe-service.yml
                                kubectl get pods
                                kubectl get services
                            """
                        }
                    }
                }
            }
        }
    }
    
    post {
        always {
            echo "Cleaning up any remaining resources"
            // Add any cleanup steps if needed
        }
    }
}
