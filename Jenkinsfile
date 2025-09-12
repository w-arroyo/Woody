pipeline {
    agent any
    triggers {
        githubPush()
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                bat 'gradle build -x test'
            }
        }

        stage('Test') {
            steps {
                bat 'gradle test'
            }
            post {
                always {
                    junit '**/build/test-results/**/*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        bat """
                        gradle sonarqube ^
                          -Dsonar.projectKey=Woody ^
                          -Dsonar.host.url=%SONAR_HOST_URL% ^
                          -Dsonar.login=%SONAR_TOKEN%
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
    post {
        always {
            cleanWs()
            echo 'Pipeline completed - cleaning workspace'
        }
        success {
            echo 'Build successful.'
        }
        failure {
            echo 'Build failed. Check tests or Quality Gates analysis.'
        }
    }
}
