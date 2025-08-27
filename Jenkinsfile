pipeline {
    agent {
        docker {
            image 'mcr.microsoft.com/dotnet/sdk:8.0'
            args '-u root:root'
        }
    }
    environment {
        DOTNET_CLI_TELEMETRY_OPTOUT = 'true'
        DOTNET_NOLOGO = 'true'
        NUGET_PACKAGES = "${WORKSPACE}/.nuget/packages"
        PATH = "${PATH}:/usr/share/dotnet"
    }
    
    stages {
        // Prerequisites, Checkout, Restore, Build, Test, Package
        // таны өмнөх pipeline stage-уудыг яг хэвээр оруулна
    }

    post {
        always {
            echo '🧹 Workspace цэвэрлэж байна...'
            sh 'rm -rf ./publish ./TestResults ./.nuget || true'
            sh 'git clean -fdx || true'
        }
    }
}
