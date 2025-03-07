pipeline {
    agent any

    tools {
        // Define la herramienta .NET SDK que has configurado en Jenkins
        dotnet 'dotnet-sdk'
    }

    environment {
        // Configura las variables de entorno para SonarQube
        SONARQUBE_SCANNER_HOME = tool 'SonarQube Scanner'
        SONAR_HOST_URL = 'http://3.85.20.213:9000'
        SONAR_LOGIN = credentials('squ_ecd4e92d760b017acf67cb824ab4b2f193a0dc7d')
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
                sh '''
                wget https://dot.net/v1/dotnet-install.sh
                chmod +x dotnet-install.sh
                ./dotnet-install.sh --version latest
                export PATH=$PATH:$HOME/.dotnet
                '''
            }
        }

        stage('Build') {
            steps {
                // Construye el proyecto .NET
                sh 'dotnet build'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                // Ejecuta el análisis de SonarQube
                withSonarQubeEnv('SonarQube') {
                    sh "${SONARQUBE_SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectKey=tu-proyecto -Dsonar.sources=. -Dsonar.host.url=${SONAR_HOST_URL} -Dsonar.login=${SONAR_LOGIN}"
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