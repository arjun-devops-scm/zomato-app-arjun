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
  }
}
