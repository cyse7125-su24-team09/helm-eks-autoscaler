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
          sh '''
          npm install @semantic-release/git
          npm install @semantic-release/exec
          npm install semantic-release-helm 
          npx semantic-release
          '''
        script {
              env.RELEASE_VERSION = readFile('.release-version').trim()
           }
      }
    } 
    stage('Mirror Images') {
    when {
          branch 'main'
          }
              steps {
                withDockerRegistry([credentialsId: 'dockerHub', url: ""]) {
                    withCredentials([usernamePassword(credentialsId: 'dockerHub', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                        echo 'Creating a multi-arch builder...'
                        sh "docker buildx create --name multiarch --use"

                        echo 'Building and publishing app images....'
                        sh """
                        docker buildx build --push --platform linux/amd64,linux/arm64 -f Dockerfile.cluster-autoscaler --no-cache \
                        -t $DOCKERHUB_USERNAME/eks-autoscaler:latest \
                        -t $DOCKERHUB_USERNAME/eks-autoscaler:${env.RELEASE_VERSION} \
                        .
                        """
                        echo 'Building and publishing flyway db migration images....'
                        sh """
                        docker buildx build --push --platform linux/amd64,linux/arm64 -f Dockerfile.metrics-server --no-cache \
                        -t $DOCKERHUB_USERNAME/metrics-server:latest \
                        -t $DOCKERHUB_USERNAME/metrics-server:${env.RELEASE_VERSION} \
                        .
                        """
                    }
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
      sh 'docker system prune -a -f'
    }
  }
}
