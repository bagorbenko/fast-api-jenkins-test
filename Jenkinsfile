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
                . $VENV/bin/activate
                pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                . $VENV/bin/activate
                PYTHONPATH=$WORKSPACE pytest --junitxml=report.xml
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
