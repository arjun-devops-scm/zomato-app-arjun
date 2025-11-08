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
 }
}
