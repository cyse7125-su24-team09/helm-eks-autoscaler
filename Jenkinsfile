pipeline {
  agent any
  tools { nodejs "node" }
  environment {
    GH_TOKEN = credentials('github-pat')
  }
  stages {
    stage('Initialize') {
      steps {
        cleanWs()

        echo 'Cloning repository...'
        checkout scm
      }
    }
    stage('Validate Commits') {
      steps {

        script {
          sh '''
          npm install @commitlint/config-conventional @commitlint/cli
          '''
          def commitLintStatus = sh(script: 'npx commitlint --from=HEAD~1 --to=HEAD', returnStatus: true)
          if (commitLintStatus != 0) {
            currentBuild.result = 'FAILURE'
            error('Conventional message check failed')
          }
        }
      }
    }    
    stage('Lint Helm Chart') {
      steps {
        script {
          def lintStatus = sh(script: 'helm lint .', returnStatus: true)
          if (lintStatus != 0) {
            currentBuild.result = 'FAILURE'
            error('Helm lint failed')
          }
        }
      }
    }
    stage('Validate Helm Template') {
      steps {
        script {
          def templateStatus = sh(script: 'helm template .', returnStatus: true)
          if (templateStatus != 0) {
            currentBuild.result = 'FAILURE'
            error('Helm template validation failed')
          }
        }
      }
    }
    stage('Release Helm Chart') {
      when {
        branch 'main'
      }
      steps {
        script {
          sh '''
          npm install @semantic-release/changelog
          npm install @semantic-release/git
          npm install @semantic-release/exec
          npm install semantic-release-helm 
          npx semantic-release
          '''
        }
      }
    }    
  }
  post {
    success {
      echo 'Linting, template validation and semantic release succeeded'
    }
    failure {
      echo 'Linting, template validation, or semantic release failed'
    }
    always {
      echo 'Pipeline complete'
    }
  }
}
