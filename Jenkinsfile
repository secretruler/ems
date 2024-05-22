pipeline {
    agent any

    stages {
        stage('CODE QUALITY CHECK') {
            steps {
                echo 'Checking..'
                echo " code quality checking......"
                sh 'sudo docker run --rm -e SONAR_HOST_URL="http://3.7.70.197:9000//" -e SONAR_LOGIN="sqp_69f7f05dc8fc7986c24018eea330e0c7d07ffbc8" -v ".:/usr/src" sonarsource/sonar-scanner-cli -Dsonar.projectKey=lms'
                echo " testing completed "
            }
        }
        stage('Build') {
            steps {
                echo 'Building..'
                sh 'cd webapp && npm install &&  npm run build'
                echo 'Building completed'
            }
        }
        stage('Read JSON') {
            steps {
                def packageJSON = readJSON file: 'webapp/package.json'
                def packageJSONVersion = packageJSON.version
                echo "${packageJSONVersion}"
                echo " Testing completed"
                
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}





