pipeline {
  agent any
  stages {
    stage('GetUrl') {
      steps {
        sh 'echo "test"'
        bat(script: 'echo "aa"', encoding: 'utf-8', label: 'master', returnStatus: true, returnStdout: true)
      }
    }

  }
}