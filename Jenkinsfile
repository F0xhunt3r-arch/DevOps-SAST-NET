pipeline {
    agent any

    environment {
        // Configura las variables de entorno para SonarQube
        SONARQUBE_SCANNER_HOME = tool name: 'SonarQube Scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
        SONAR_HOST_URL = 'http://3.85.20.213:9000'
        SONAR_LOGIN = 'squ_ecd4e92d760b017acf67cb824ab4b2f193a0dc7d'
    }

    stages {
        stage('Checkout') {
            steps {
                // Clona el repositorio usando las credenciales de GitHub
                git url: 'https://github.com/F0xhunt3r-arch/DevOps-SAST-NET.git', credentialsId: 'github-token'
            }
        }

        stage('Install .NET SDK') {
            steps {
                // Instala el .NET SDK
                bat '''
                curl -L -o dotnet-install.ps1 https://dot.net/v1/dotnet-install.ps1
                powershell -NoProfile -ExecutionPolicy Bypass -File dotnet-install.ps1 -Version latest
                set PATH=%PATH%;%USERPROFILE%\\.dotnet
                '''
            }
        }

        stage('Build') {
            steps {
                // Construye el proyecto .NET
                bat 'dotnet build'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                // Ejecuta el análisis de SonarQube
                withSonarQubeEnv('SonarQube') {
                    bat 'dotnet sonarscanner begin /k:"DevOps-SAST-NET" /d:sonar.host.url=%SONAR_HOST_URL% /d:sonar.login=%SONAR_LOGIN%'
                    bat 'dotnet build'
                    bat 'dotnet sonarscanner end /d:sonar.login=%SONAR_LOGIN%'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                // Espera a que SonarQube termine el análisis
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}