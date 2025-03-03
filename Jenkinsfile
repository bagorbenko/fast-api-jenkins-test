pipeline {
    agent { label 'gcp' }

    environment {
        DEPLOY_DIR = "/var/lib/jenkins/deploy"
    }

    stages {
        stage('Checkout') {
            steps {
                cleanWs()
                git branch: 'main', url: 'https://github.com/bagorbenko/fast-api-jenkins-test.git'
            }
        }

        stage('Setup Python Environment') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install -r requirements.txt
                    pip install pytest pytest-html
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                    . venv/bin/activate
                    mkdir -p reports
                    PYTHONPATH=. pytest --junitxml=reports/report.xml --html=reports/report.html --self-contained-html || true
                '''
            }
            post {
                always {
                    junit 'reports/report.xml'
                    archiveArtifacts artifacts: 'reports/**', allowEmptyArchive: true
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    echo "Deploy step - ничего не делаем, просто для примера"
                '''
            }
        }
    }

    post {
        success {
            sh """
                curl -X POST "https://api.telegram.org/bot7511855444:AAEDvkMdddaKa4B2AArcudj7IEzQUQF6Lm8/sendMessage" \\
                    -H "Content-Type: application/json" \\
                    -d '{
                        "chat_id": "269199712",
                        "text": "✅ Обновление билда: УДАЧНО\\nПроект: ${env.JOB_NAME}\\nБилд номер: ${env.BUILD_NUMBER}\\nСсылка: ${env.BUILD_URL}",
                        "disable_notification": false
                    }'

                if [ -f "reports/report.html" ]; then
                    curl -X POST "https://api.telegram.org/bot7511855444:AAEDvkMdddaKa4B2AArcudj7IEzQUQF6Lm8/sendDocument" \\
                        -F "chat_id=269199712" \\
                        -F "document=@reports/report.html" \\
                        -F "caption=Тестовый отчет для билда ${env.BUILD_NUMBER} - УДАЧНО"
                fi
            """
        }

        failure {
            sh """
                curl -X POST "https://api.telegram.org/bot7511855444:AAEDvkMdddaKa4B2AArcudj7IEzQUQF6Lm8/sendMessage" \\
                    -H "Content-Type: application/json" \\
                    -d '{
                        "chat_id": "269199712",
                        "text": "❌ Обновление билда: ОШИБКА\\nПроект: ${env.JOB_NAME}\\nБилд номер: ${env.BUILD_NUMBER}\\nСсылка: ${env.BUILD_URL}",
                        "disable_notification": false
                    }'

                if [ -f "reports/report.html" ]; then
                    curl -X POST "https://api.telegram.org/bot7511855444:AAEDvkMdddaKa4B2AArcudj7IEzQUQF6Lm8/sendDocument" \\
                        -F "chat_id=269199712" \\
                        -F "document=@reports/report.html" \\
                        -F "caption=Тестовый отчет для билда ${env.BUILD_NUMBER} - ОШИБКА"
                fi
            """
        }
    }
}
