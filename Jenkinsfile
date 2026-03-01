pipeline {
    agent any
    
    tools {
        jdk 'jdk22'
        maven 'maven3'
    }

    stages {
        stage('Initialize') {
            steps {
                echo 'Khởi tạo hệ thống CI...'
                sh 'mvn --version'
            }
        }

        stage('Customer Service') {
            // when { changeset "customer/**" }
            steps {
                echo 'Có thay đổi ở Customer Service...'
                sh 'mvn clean install -pl customer -am'
            }
            post {
                always {
                    jacoco(minimumInstructionCoverage: '70')
                }
            }
        }

        stage('Static Code Analysis (SonarCloud)') {
            // when { changeset "customer/**" }
            steps {
                echo 'Đang quét bảo mật với SonarCloud...'
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    // Dùng -DskipTests để không phải chạy lại Test một lần nữa, tiết kiệm RAM
                    sh "mvn sonar:sonar -Dsonar.token=${SONAR_TOKEN} -pl customer -am -DskipTests"
                }
            }
        }
    }
}