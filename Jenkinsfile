pipeline {
    agent any
    tools {
        gradle 'gradle'
    }
    triggers {
        githubPush()     // automatic trigger
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                sh './gradlew build -x test'
            }
        }
        stage('Test') {
            steps {
                sh './gradlew test'
            }
            post {
                always {
                    junit 'build/test-results/**/*.xml'
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    sh './gradlew sonarqube'
                }
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
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
            echo 'Build successful. Code completed all tests and quality gates.'
        }
        failure {
            echo 'Build unsuccessful. Check quality gates or tests'
        }
    }
}