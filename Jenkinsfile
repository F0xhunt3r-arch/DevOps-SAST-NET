pipeline {
    agent any

    environment {
        // variables de entorno para SonarQube
        SONARQUBE_SCANNER_HOME = "${WORKSPACE}/sonar-scanner"
        SONAR_HOST_URL = 'http://13.217.163.150:9000/'
        SONAR_LOGIN = credentials('Token-SonarQube')
        PATH = "${env.PATH};C:\\Windows\\System32;C:\\Program Files\\Git\\bin;C:\\Windows\\System32\\WindowsPowerShell\\v1.0;C:\\Program Files\\dotnet"
    }

    stages {
        stage('Checkout') {
            steps {
                // credenciales de GitHub
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], userRemoteConfigs: [[url: 'https://github.com/F0xhunt3r-arch/DevOps-SAST-NET.git', credentialsId: 'github-token-NET']]])
            }
        }

        stage('Print PATH') {
            steps {
                // Imprime la variable de entorno PATH
                bat 'echo %PATH%'
            }
        }

        stage('Install .NET SDK') {
            steps {
                // Instala el .NET SDK
                bat '''
                curl -L -o dotnet-install.ps1 https://dot.net/v1/dotnet-install.ps1
                powershell -NoProfile -ExecutionPolicy Bypass -File dotnet-install.ps1 -Version latest
                set PATH=%PATH%;%USERPROFILE%\\.dotnet\\
                '''
            }
        }

        stage('Install SonarQube Scanner') {
            steps {
                // Descarga e instala SonarQube Scanner
                bat '''
                curl -L -o sonar-scanner.zip https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-5.0.1.3006-windows.zip
                powershell -NoProfile -ExecutionPolicy Bypass -Command "Expand-Archive -Path sonar-scanner.zip -DestinationPath . -Force"
                if exist sonar-scanner rmdir /s /q sonar-scanner
                powershell -NoProfile -ExecutionPolicy Bypass -Command "Get-ChildItem -Path . -Filter 'sonar-scanner-*' | Rename-Item -NewName 'sonar-scanner'"
                set PATH=%PATH%;%WORKSPACE%\\sonar-scanner\\bin
                '''
            }
        }

        stage('Clean') {
            steps {
                // Limpia el directorio obj
                bat 'rmdir /s /q obj'
            }
        }

        stage('Build') {
            steps {
                script {
                    // Construye el proyecto .NET y captura el estado de salida
                    def buildStatus = bat(returnStatus: true, script: 'dotnet build')
                    if (buildStatus != 0) {
                        echo "Build failed with status: ${buildStatus}, but continuing with SonarQube analysis."
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                // Ejecuta el análisis de SonarQube
                withSonarQubeEnv('SonarQube') {
                    bat "${env.SONARQUBE_SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectKey=DevOps-SAST-NET -Dsonar.host.url=%SONAR_HOST_URL% -Dsonar.login=%SONAR_LOGIN%"
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 10, unit: 'MINUTES') {
                        def startTime = System.currentTimeMillis()
                        def qg = waitForQualityGate()
                        while (qg.status == 'PENDING') {
                            sleep 10
                            def elapsedTime = (System.currentTimeMillis() - startTime) / 1000
                            echo "Tiempo transcurrido: ${elapsedTime} segundos"
                            qg = waitForQualityGate()
                        }
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }
    }
}
