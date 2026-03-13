pipeline {
    agent any

    environment {
        SONARQUBE = 'sonarqube-server'
        REGISTRY = 'ghcr.io/mercy299/transformation-dictionary'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Lint') {
            steps {
                sh 'npm run lint'
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv("${SONARQUBE}") {
                    sh """
                        sonar-scanner \
                        -Dsonar.projectKey=node-project \
                        -Dsonar.projectName=node-project \
                        -Dsonar.sources=. \
                        -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info
                    """
                }
            }
        }

        stage("Build Production") {
            steps {
                sh 'npm run build'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Extract version from package.json
                    def version = sh(
                        script: "node -p \"require('./package.json').version\"",
                        returnStdout: true
                    ).trim()

                    // GHCR Login (uses Jenkins Credentials)
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'Jenkins-Test',
                            usernameVariable: 'GHCR_USER',
                            passwordVariable: 'GHCR_TOKEN'
                        )
                    ]) {
                        sh """
                            echo $GHCR_TOKEN | docker login ghcr.io -u $GHCR_USER --password-stdin
                        """
                    }

                    // Build docker image using REGISTRY from Jenkins UI
                    sh "docker build -t ${env.REGISTRY}:${version}-${env.BUILD_NUMBER} ."
                }
            }
        }

        stage('Push Docker Image') {
            when {
                expression { return env.BRANCH_NAME == 'main' }
            }
            steps {
                script {
                    def version = sh(
                        script: "node -p \"require('./package.json').version\"",
                        returnStdout: true
                    ).trim()

                    
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'Jenkins-Test',
                            usernameVariable: 'GHCR_USER',
                            passwordVariable: 'GHCR_TOKEN'
                        )
                    ]) {
                        sh """
                            echo $GHCR_TOKEN | docker login ghcr.io -u $GHCR_USER --password-stdin
                        """
                    }

                    sh "docker push ${env.REGISTRY}:${version}-${env.BUILD_NUMBER}"
                }
            }
        }
    }
}
