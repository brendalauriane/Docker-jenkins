pipeline {
    agent any

    environment {
        CONTAINER_ID   = ''
        SUM_PY_PATH    = "${WORKSPACE}\\sum.py"
        DIR_PATH       = "${WORKSPACE}"
        TEST_FILE_PATH = "${WORKSPACE}\\test_variables.txt"
    }

    stages {
        stage('Build') {
            steps {
                bat "docker build -t sum-python-app \"%DIR_PATH%\""
            }
        }
    }
}