pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check Node and npm') {
            steps {
                bat '''
                node -v
                call npm -v
                '''
            }
        }

        stage('Install dependencies') {
            steps {
                bat '''
                call npm install
                '''
            }
        }

        stage('Start app') {
            steps {
                bat '''
                powershell -Command "Get-NetTCPConnection -LocalPort 3032 -ErrorAction SilentlyContinue | ForEach-Object { Stop-Process -Id $_.OwningProcess -Force -ErrorAction SilentlyContinue }"

                powershell -Command "Start-Process -FilePath node -ArgumentList 'index.js','3032' -WorkingDirectory '%CD%' -WindowStyle Hidden"
                '''
            }
        }

        stage('Wait for app') {
            steps {
                bat '''
                powershell -Command "$deadline=(Get-Date).AddSeconds(30); while((Get-Date) -lt $deadline) { try { $response = Invoke-WebRequest -UseBasicParsing http://localhost:3032 -TimeoutSec 2; if ($response.StatusCode -eq 200) { exit 0 } } catch { Start-Sleep -Seconds 1 } }; exit 1"
                '''
            }
        }

        stage('Run tests') {
            steps {
                bat '''
                call npm test
                '''
            }
        }
    }

    post {
        always {
            bat '''
            powershell -Command "Get-NetTCPConnection -LocalPort 3032 -ErrorAction SilentlyContinue | ForEach-Object { Stop-Process -Id $_.OwningProcess -Force -ErrorAction SilentlyContinue }"
            '''
        }
    }
}