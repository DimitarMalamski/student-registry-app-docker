pipeline {
    agent any

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

        stage('Start app and run tests') {
            steps {
                bat '''
                powershell -Command "Get-NetTCPConnection -LocalPort 3032 -ErrorAction SilentlyContinue | ForEach-Object { Stop-Process -Id $_.OwningProcess -Force -ErrorAction SilentlyContinue }"

                powershell -Command "Start-Process -FilePath node -ArgumentList 'index.js','3032' -WorkingDirectory '%CD%' -WindowStyle Hidden"

                powershell -Command "Start-Sleep -Seconds 10"

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