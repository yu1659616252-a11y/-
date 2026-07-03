pipeline {
    agent any

    parameters {
        string(name: 'API_BASE_URL', defaultValue: 'mock', description: 'Target API base URL, use mock for demo')
        string(name: 'PYTEST_MARK', defaultValue: 'smoke', description: 'Pytest marker, for example: smoke or api')
    }

    environment {
        VENV_DIR = '.venv'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Prepare Python') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'python3 -m venv ${VENV_DIR}'
                        sh '. ${VENV_DIR}/bin/activate && python -m pip install --upgrade pip'
                        sh '. ${VENV_DIR}/bin/activate && pip install -r requirements.txt'
                    } else {
                        bat 'python -m venv %VENV_DIR%'
                        bat '%VENV_DIR%\\Scripts\\python -m pip install --upgrade pip'
                        bat '%VENV_DIR%\\Scripts\\pip install -r requirements.txt'
                    }
                }
            }
        }

        stage('Run API Tests') {
            steps {
                script {
                    if (isUnix()) {
                        sh '. ${VENV_DIR}/bin/activate && pytest -m "${PYTEST_MARK}" --base-url="${API_BASE_URL}"'
                    } else {
                        bat '%VENV_DIR%\\Scripts\\pytest -m "%PYTEST_MARK%" --base-url="%API_BASE_URL%"'
                    }
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'allure-results/**', allowEmptyArchive: true
                }
            }
        }
    }

    post {
        always {
            allure includeProperties: false, jdk: '', results: [[path: 'allure-results']]
        }
    }
}
