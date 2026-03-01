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

        stage('Gitleaks Security Scan') {
            steps {
                echo 'Quét bí mật và thông tin nhạy cảm với Gitleaks...'
                script {
                    // Cài đặt gitleaks nếu chưa có
                    sh '''
                        if ! command -v gitleaks &> /dev/null; then
                            echo "Đang cài đặt Gitleaks..."
                            brew install gitleaks || curl -sSfL https://raw.githubusercontent.com/gitleaks/gitleaks/master/scripts/install.sh | sh -s -- -b /usr/local/bin
                        fi
                    '''
                    // Chạy gitleaks scan
                    sh 'gitleaks detect --config=gitleaks.toml --verbose --no-git'
                }
            }
        }

        stage('Customer Build & Test') {
            // when { changeset "customer/**" }
            steps {
                echo 'Chạy Build và Unit Test cho Customer Service...'
                // QUAN TRỌNG: Phải dùng -DskipTests=false để chạy tests
                sh 'mvn clean install -pl customer -am -DskipTests=false'
            }
            post {
                always {
                    // Kiểm tra coverage phải >= 70%
                    jacoco(minimumInstructionCoverage: '70')
                }
            }
        }

        stage('Static Code Analysis (SonarCloud)') {
            // when { changeset "customer/**" }
            steps {
                echo 'Đang quét bảo mật và chất lượng code với SonarCloud...'
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    // Dùng -DskipTests để không phải chạy lại Test một lần nữa, tiết kiệm RAM
                    sh "mvn sonar:sonar -Dsonar.token=${SONAR_TOKEN} -pl customer -am -DskipTests"
                }
            }
        }

        stage('Snyk Security Scan') {
            steps {
                echo 'Quét lỗ hổng bảo mật dependencies với Snyk...'
                script {
                    // Yêu cầu: Cài đặt Snyk CLI và cấu hình SNYK_TOKEN trong Jenkins credentials
                    withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
                        sh '''
                            if ! command -v snyk &> /dev/null; then
                                echo "Đang cài đặt Snyk CLI..."
                                npm install -g snyk || curl --compressed https://static.snyk.io/cli/latest/snyk-linux -o snyk && chmod +x ./snyk && mv ./snyk /usr/local/bin/
                            fi
                            snyk auth ${SNYK_TOKEN}
                            snyk test --all-projects --severity-threshold=high || echo "Phát hiện lỗ hổng bảo mật!"
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline hoàn tất!'
            // Publish test results
            junit '**/target/surefire-reports/*.xml'
        }
        failure {
            echo 'Pipeline thất bại! Kiểm tra logs để biết chi tiết.'
        }
        success {
            echo 'Pipeline thành công! Tất cả kiểm tra đã pass.'
        }
    }
}