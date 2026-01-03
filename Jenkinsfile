pipeline {
  agent any

  stages {

    stage('Build') {
      agent {
        docker {
          image 'maven:3.9.12-eclipse-temurin-17'
          reuseNode true
        }
      }
      steps {
        echo 'Compile Maven app'
        sh 'mvn compile'
      }
    }

    stage('Test') {
      agent {
        docker {
          image 'maven:3.9.12-eclipse-temurin-17'
          reuseNode true
        }
      }
      steps {
        echo 'Test Maven app'
        sh 'mvn test'
      }
    }

    stage('Package & Docker') {
      parallel {

        stage('Maven Package') {
          agent {
            docker {
              image 'maven:3.9.12-eclipse-temurin-17'
              reuseNode true
            }
          }
          steps {
            echo 'Package Maven app'

            sh '''
              GIT_SHORT_COMMIT=$(echo $GIT_COMMIT | cut -c 1-7)
              mvn versions:set -DnewVersion="$GIT_SHORT_COMMIT"
              mvn versions:commit
              mvn package -DskipTests
            '''

            archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
          }
        }

        stage('Docker Build & Push') {
          agent any
          steps {
            script {
              docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                def commitHash = env.GIT_COMMIT.take(7)
                def image = docker.build("hoangquan2022/sysfoo:${commitHash}")
                image.push()
                image.push('latest')
                image.push('dev')
              }
            }
          }
        }
      }
    }

    stage('Deploy') {
      agent any
      steps {
        sh '''
          docker stop sysfoo || true
          docker rm sysfoo || true
          docker run -d -p 8954:8080 --name sysfoo hoangquan2022/sysfoo:latest
        '''
      }
    }
  }

  post {
    always {
      echo 'Pipeline completed'
    }
  }
}
