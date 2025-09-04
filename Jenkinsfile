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
                checkout scm
            }
        }

        stage('Static checks & Unit tests') {
            steps {
                bat 'python -m pip install --upgrade pip'
                bat 'python -m pip install -r requirements-dev.txt'

                // run flake8 but don't fail build on warnings
                bat 'flake8 . || exit 0'

                // ensure test-results directory exists
                bat 'if not exist test-results mkdir test-results'

                // run pytest and generate XML
                bat 'python -m pytest -q --disable-warnings --junitxml=test-results\\pytest.xml'
                
                // confirm pytest.xml exists
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
                echo "Building Docker..."
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
