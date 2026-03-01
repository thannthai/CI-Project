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
            steps {
                echo 'Đang quét bảo mật với SonarCloud...'
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    // Nhớ thay thế giá trị tổ chức và project key của bạn vào đây
                    sh "mvn sonar:sonar -Dsonar.token=${SONAR_TOKEN} -Dsonar.organization=thannthai -Dsonar.projectKey=thannthai_CI-Project -pl customer -am -DskipTests"
                }
            }
        }
    }
}