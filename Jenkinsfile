pipeline {
  agent {
    docker {
      image 'mcr.microsoft.com/dotnet/sdk:8.0'
      args '-u root'
    }
  }

  environment {
    DOTNET_CLI_TELEMETRY_OPTOUT = 'true'
    DOTNET_NOLOGO = 'true'
    NUGET_PACKAGES = "${WORKSPACE}/.nuget/packages"
  }

  options {
    timestamps()
    ansiColor('xterm')
  }

  triggers {
    // GitHub webhook тохируулсан бол Jenkins multibranch/SCM polling ашиглаж болно.
    // pollSCM('H/2 * * * *')
  }

  stages {
    stage('Checkout') {
      when { expression { env.BRANCH_NAME == 'feature-ci-pipeline' || env.GIT_BRANCH == 'origin/feature-ci-pipeline' } }
      steps {
        checkout([$class: 'GitSCM',
          branches: [[name: '*/feature-ci-pipeline']],
          userRemoteConfigs: [[url: 'https://github.com/<your-user>/<your-fork>.git', credentialsId: 'jenkins-ci-token']]
        ])
        sh 'git log -1 --oneline'
      }
    }

    stage('Restore') {
      steps { sh 'dotnet restore --verbosity minimal' }
    }

    stage('Build') {
      steps { sh 'dotnet build --configuration Release --no-restore --verbosity minimal' }
    }

    stage('Test - All') {
      steps {
        sh '''
          mkdir -p TestResults
          dotnet test --configuration Release --no-build \
            --logger "trx;LogFileName=AllTests.trx" \
            --results-directory ./TestResults
        '''
      }
      post {
        always {
          script {
            if (fileExists('TestResults')) {
              archiveArtifacts artifacts: 'TestResults/**', fingerprint: true, allowEmptyArchive: true
            }
          }
          junit allowEmptyResults: true, testResults: 'TestResults/**/*.trx'
        }
      }
    }

    stage('Publish') {
      when { expression { currentBuild.currentResult == 'SUCCESS' } }
      steps {
        sh 'dotnet publish --configuration Release --no-build --output ./publish'
        sh 'ls -la ./publish'
      }
      post {
        success {
          archiveArtifacts artifacts: 'publish/**', fingerprint: true, allowEmptyArchive: true
        }
      }
    }
  }

  post {
    success { echo '🎉 Pipeline SUCCESS' }
    unstable { echo '⚠️ Pipeline UNSTABLE' }
    failure { echo '💥 Pipeline FAILED' }
    always {
      sh 'rm -rf ./publish || true'
      sh 'rm -rf ./TestResults || true'
      sh 'rm -rf ./.nuget || true'
      sh 'git clean -fdx || true'
    }
  }
}
