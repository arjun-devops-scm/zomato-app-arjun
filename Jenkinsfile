pipeline { 
  agent {
   label "jenkins-agent"
}
tools {
  nodejs 'nodejs-23'
}
stages {
  stage ('Installing dependencies') {
    steps {
      script {
        sh "npm install"
      }
    }
  }
  stage ('test') {
   steps {
      script {
        sh "npm run test"
    }
   }
  }
  stage ('Trivy file system scan') {
    steps {
      script {
        sh "trivy fs --format json -o trivy-files-scan-report.json ."
        archiveArtifacts artifacts: 'trivy-files-scan-report.json', fingerprint: true
      }
    }
  }
  stage ('sonar analysis') {
    steps {
      script {
        def SONAR_SCANNER_HOME = tool name: 'sonar-scanner'
        withSonarQubeEnv('sonar') {
          sh "${SONAR_SCANNER_HOME}/bin/sonar-scanner -Dsonar.lanague=nodejs -Dsonar.projectKey=zomoto"
        }
        }
      }
    }
    stage ('sonar quality gate') {
      steps {
        script {
          waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
        }
      }
    }
    stage ('docker build') {
      steps {
        script {
          sh "docker build -t arjundocker92/zomato:${BUILD_NUMBER} ."
        }
      }
    }
   stage ('Trivy scan image') {
     steps {
       script {
         sh "trivy image --format json -o zomate-image-trivy-scan-report.json arjundocker92/zomato:${BUILD_NUMBER}"
         archiveArtifacts artifacts: 'zomate-image-trivy-scan-report.json',  fingerprint: true
       }
     }
     stage ('Pushing Image to Docker hub') {
       steps {
         script {
           withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'docker_user', passwordVariable: 'docker_password')]) {
             echo '$docker_password | docker login -u $docker_user --password-stdin'
             sh "docker push arjundocker92/zomato:${BUILD_NUMBER}"
           }
         }
       }
     }
     stage ('deploy zomato app') {
       steps {
         script {
           sshagent (credentials: ['deploy-sever-creds']) {
             def SERVER_IP = "65.0.68.100"
              sh """
                  ssh -o StrictHostKeyChecking=no root@${SERVER_IP} \
                  'echo "Pulling image..."; \
                  docker pull arjundocker92/zomato:${BUILD_NUMBER}; \
                  docker stop zomato || true; \
                  docker rm zomato || true; \
                  docker run -d --name zomato -p 3000:3000 arjundocker92/zomato:${BUILD_NUMBER}'
              """
          }
         }
       }
     }
  }
}
