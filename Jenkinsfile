pipeline {
  agent any
  stages {
    stage('build') {
      agent {
        docker {
          image 'maven:3.9.12-eclipse-temurin-17'
        }

      }
      steps {
        echo 'compile maven app'
        sh 'mvn compile'
      }
    }

    stage('test') {
      agent {
        docker {
          image 'maven:3.9.12-eclipse-temurin-17'
        }

      }
      steps {
        echo 'test maven app'
        sh 'mvn clean test'
      }
    }

    stage('package') {
      parallel {
        stage('package') {
          agent {
            docker {
              image 'maven:3.9.12-eclipse-temurin-17'
            }

          }
          steps {
            echo 'package maven app'
            sh '''# Truncate the GIT_COMMIT to the first 7 characters
GIT_SHORT_COMMIT=$(echo $GIT_COMMIT | cut -c 1-7)
# Set the version using Maven
mvn versions:set -DnewVersion="$GIT_SHORT_COMMIT"
mvn versions:commit'''
            sh 'mvn package -DskipTests'
            archiveArtifacts '**/target/*.jar'
          }
        }

        stage('Docker B&P') {
          agent any
          steps {
            script {
              docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                def commitHash = env.GIT_COMMIT.take(7)
                def dockerImage = docker.build("hoangquan2022/sysfoo:${commitHash}", "./")
                dockerImage.push()
                dockerImage.push("latest")
                dockerImage.push("dev")
              }
            }

          }
        }

      }
    }

    stage('Deploy') {
      steps {
        sh '''docker stop sysfoo || true
docker rm sysfoo || true
docker run -d -p 8954:8080 --name sysfoo sysfoo:latest'''
      }
    }

  }
  tools {
    maven 'Maven 3.9.12'
  }
  post {
    always {
      echo 'This pipeline is completed..'
    }

  }
}