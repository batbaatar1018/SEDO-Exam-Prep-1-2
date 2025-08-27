pipeline {
    agent any
    
    environment {
        // .NET CLI телеметри идэвхгүйжүүлэх
        DOTNET_CLI_TELEMETRY_OPTOUT = 'true'
        DOTNET_NOLOGO = 'true'
        // NuGet пакет кэш workspace дотор хадгалах
        NUGET_PACKAGES = "${WORKSPACE}/.nuget/packages"
        // .NET PATH-д нэмэх
        PATH = "${PATH}:/usr/share/dotnet"
    }
    
    stages {
        stage('Prerequisites Check') {
            steps {
                echo '🔍 Шаардлагатай зүйлсийг шалгаж байна...'
                script {
                    try {
                        sh 'dotnet --version'
                        echo '✅ .NET SDK амжилттай олдлоо'
                    } catch (Exception e) {
                        error "❌ .NET SDK олдсонгүй: ${e.getMessage()}"
                    }
                }
            }
        }
        
        stage('Checkout') {
            steps {
                echo '📥 Git repository-оос код татаж байна...'
                git branch: 'feature-ci-pipeline',
                    url: 'https://github.com/TgEsEvo/exam-prep-2.git',
                    credentialsId: 'jenkins-ci-token'
                
                echo '✅ Checkout амжилттай'
                // Файлын жагсаалт харах
                sh 'ls -la'
            }
        }
        
        stage('Restore') {
            steps {
                echo '📦 NuGet пакетуудыг restore хийж байна...'
                script {
                    try {
                        sh 'dotnet restore --verbosity normal'
                        echo '✅ Restore амжилттай'
                    } catch (Exception e) {
                        error "❌ Restore алдаа: ${e.getMessage()}"
                    }
                }
            }
        }
        
        stage('Build') {
            steps {
                echo '🔨 Project-ийг build хийж байна...'
                script {
                    try {
                        sh 'dotnet build --configuration Release --no-restore --verbosity normal'
                        echo '✅ Build амжилттай'
                    } catch (Exception e) {
                        error "❌ Build алдаа: ${e.getMessage()}"
                    }
                }
            }
        }
        
        stage('Test') {
            steps {
                echo '🧪 Тестүүд ажиллуулж байна...'
                script {
                    try {
                        sh '''
                            dotnet test \
                                --configuration Release \
                                --no-build \
                                --verbosity normal \
                                --logger "trx;LogFileName=TestResults.trx" \
                                --results-directory ./TestResults \
                                --collect:"XPlat Code Coverage"
                        '''
                        echo '✅ Бүх тестүүд амжилттай'
                    } catch (Exception e) {
                        echo "⚠️ Зарим тестүүд алдаатай: ${e.getMessage()}"
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
            post {
                always {
                    script {
                        // Test үр дүнг архивлах
                        try {
                            sh 'ls -la TestResults/ || echo "TestResults хавтас байхгүй"'
                            
                            if (sh(script: 'ls TestResults/*.trx 2>/dev/null', returnStatus: true) == 0) {
                                echo '📊 Test үр дүнг архивлаж байна...'
                                archiveArtifacts artifacts: 'TestResults/*.trx', 
                                               fingerprint: true, 
                                               allowEmptyArchive: true
                            } else {
                                echo '📊 Test results олдсонгүй'
                            }
                            
                            // Coverage report байгаа бол архивлах
                            if (sh(script: 'find TestResults -name "coverage.cobertura.xml" 2>/dev/null', returnStatus: true) == 0) {
                                echo '📈 Coverage report архивлаж байна...'
                                archiveArtifacts artifacts: 'TestResults/**/coverage.cobertura.xml', 
                                               fingerprint: true, 
                                               allowEmptyArchive: true
                            } else {
                                echo '📈 Coverage report олдсонгүй'
                            }
                        } catch (Exception e) {
                            echo "Архив алдаа: ${e.getMessage()}"
                        }
                    }
                }
            }
        }
        
        stage('Package') {
            when {
                expression { 
                    return currentBuild.result == null || currentBuild.result == 'SUCCESS' 
                }
            }
            steps {
                echo '📦 Application package хийж байна...'
                script {
                    try {
                        sh '''
                            dotnet publish \
                                --configuration Release \
                                --no-build \
                                --output ./publish \
                                --verbosity normal
                        '''
                        
                        // Publish хавтасны агуулгыг харах
                        sh 'ls -la ./publish'
                        
                        echo '✅ Package амжилттай'
                    } catch (Exception e) {
                        echo "⚠️ Package алдаа: ${e.getMessage()}"
                    }
                }
            }
            post {
                success {
                    // Build artifacts архивлах
                    archiveArtifacts artifacts: 'publish/**', 
                                   fingerprint: true, 
                                   allowEmptyArchive: true
                }
            }
        }
    }
    
    post {
        always {
            echo '🧹 Workspace цэвэрлэж байна...'
            script {
                try {
                    // Temp файлуудыг устгах
                    sh 'rm -rf ./publish || true'
                    sh 'rm -rf ./TestResults || true'
                    sh 'rm -rf ./.nuget || true'
                    
                    // Git clean
                    sh 'git clean -fdx || true'
                    
                    echo '✅ Cleanup амжилттай'
                } catch (Exception e) {
                    echo "⚠️ Cleanup алдаа: ${e.getMessage()}"
                }
            }
        }
        
        success {
            echo '''
            🎉 ===================================
               PIPELINE АМЖИЛТТАЙ ДУУСЛАА!
            ===================================
            ✅ Code checkout: Амжилттай
            ✅ Package restore: Амжилттай  
            ✅ Build: Амжилттай
            ✅ Test: Амжилттай
            ✅ Package: Амжилттай
            '''
        }
        
        failure {
            echo '''
            💥 ===================================
               PIPELINE АЛДААТАЙ ДУУСЛАА!
            ===================================
            ❌ Дэлгэрэнгүй мэдээллийг дээрх логоос үзнэ үү.
            '''
        }
        
        unstable {
            echo '''
            ⚠️ ===================================
              PIPELINE ТОГТВОРГҮЙ ТӨЛӨВТЭЙ!
            ===================================
            ✅ Build: Амжилттай
            ⚠️ Test: Зарим тестүүд алдаатай
            📊 Test report-ыг шалгана уу.
            '''
        }
        
        changed {
            script {
                def status = currentBuild.result ?: 'SUCCESS'
                echo "🔄 Pipeline төлөв өмнөх build-ээс өөрчлөгдлөө: ${status}"
            }
        }
    }
}