pipeline {
    agent any
    tools{
        maven 'Maven 3.9.12'
    } 
    stages {
        stage('one') {
            steps {
                echo 'compile maven app'
                sh 'mvn compile'
            }
        }

        stage('two') {
            steps {
                echo 'test maven app'
                sh 'mvn clean test'
            }
        }

        stage('three') {
            steps {
                echo 'package maven app'
                sh 'mvn package -DskipTests'
            }
        }
    }

    post {
        always {
            echo 'This pipeline is completed..'
        }
    }
}
