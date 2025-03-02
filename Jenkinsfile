pipeline {
    agent { label 'gcp' }

    environment {
        VENV = "${WORKSPACE}/venv"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/bagorbenko/fast-api-jenkins-test.git'
            }
        }

        stage('Setup Python') {
            steps {
                sh '''
                python3 -m venv $VENV
                $VENV/bin/python --version
                $VENV/bin/pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                PYTHONPATH=$WORKSPACE $VENV/bin/python -m pytest --junitxml=report.xml
                '''
            }
        }

        stage('Publish Test Report') {
            steps {
                publishHTML([allowMissing: true,
                             alwaysLinkToLastBuild: true,
                             keepAll: true,
                             reportDir: '.',
                             reportFiles: 'report.xml',
                             reportName: 'Test Report'])
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'report.xml', allowEmptyArchive: true
        }
    }
}
