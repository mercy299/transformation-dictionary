pipeline {
    agent any

    environment {
        SONARQUBE = 'sonarqube-server'
    }

    stages {

        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Install Dependencies') {
            steps { sh 'npm install' }
        }

        stage('Lint') {
            steps { sh 'npm run lint' }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv("${SONARQUBE}") {
                    sh """
                        sonar-scanner \
                        -Dsonar.projectKey=node-project \
                        -Dsonar.sources=. \
                        -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info
                    """
                }
            }
        }

        stage('Build App') {
            steps { sh 'npm run build' }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def version = sh(script: "node -p \"require('./package.json').version\"", returnStdout: true).trim()
                    sh "docker build -t $REGISTRY:${version}-${env.BUILD_NUMBER} ."
                }
            }
        }
    }
}
``
