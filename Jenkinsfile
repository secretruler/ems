pipeline {
    agent any
    stages {
        stage('Runing a Database container') {
            steps {
                echo "building a database container"
                sh 'docker container run -dt --name lms-db -e POSTGRES_PASSWORD=password postgres'
                echo 'building database container completed'
                //
            }
        }
        stage('Test') {
            steps {
                //
            }
        }
        stage('Deploy') {
            steps {
                //
            }
        }
    }
}