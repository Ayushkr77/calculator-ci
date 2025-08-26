pipeline {
    agent any
    options { 
        timestamps() 
    }
    triggers { 
        githubPush() // works if webhook is configured; else use Poll SCM
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm // if job is "Pipeline script from SCM"
            }
        }

        stage('Static checks & Unit tests') {
            steps {
                bat 'python -m pip install --upgrade pip'
                bat 'python -m pip install -r requirements-dev.txt'
                bat 'flake8 .'
                // ensure test-results directory exists
                bat 'if not exist test-results mkdir test-results'
                // run pytest and always generate report
                bat 'python -m pytest -q --disable-warnings --junitxml=test-results\\pytest.xml || exit 0'
                // list files to confirm report exists
                bat 'dir test-results'
            }
            post {
                always {
                    junit 'test-results/pytest.xml'
                }
            }
        }

        stage('Docker Build') {
            steps {
                bat "docker build -t calculator-app:%BUILD_NUMBER% ."
            }
        }

        stage('Deploy (Local Run)') {
            steps {
                bat "docker run --rm calculator-app:%BUILD_NUMBER%"
            }
        }
    }
}