pipeline { 
  agent {
   label "jenkins-agent"
}
tools {
  nodejs 'nodejs-23'
}
stages {
  stage ('test') {
   steps {
      script {
        sh "npm run test"
    }
   }
  }
 }
}
