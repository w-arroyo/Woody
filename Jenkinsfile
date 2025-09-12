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
                    withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
                        bat """
                        gradle sonar ^
                          -Dsonar.projectKey=Woody ^
                          -Dsonar.host.url=http://192.168.1.50:9000 ^
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
