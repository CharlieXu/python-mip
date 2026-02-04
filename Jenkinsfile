pipeline {
    agent any

    triggers {
        cron('H H/2 * * *')
    }

    environment {
        BRANCH_NAME = 'master'
        REPORT_DIR = 'reports'
        REPORT_FILE = 'unittest_report.html'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "*/${env.BRANCH_NAME}"]],
                    userRemoteConfigs: [[
                        url: "${env.GIT_REPO_URL}".replace('https://', "https://${env.GIT_USERNAME}:${env.GIT_PASSWORD}@")
                    ]]
                ])
            }
        }

        stage('Run Tests') {
            steps {
                bat '''
                    @echo off
                    setlocal enabledelayedexpansion
                    set "PYTHON=%VENV_PATH%\\Scripts\\python.exe"
                    if not exist "%PYTHON%" (
                        echo Python executable not found at %PYTHON%
                        exit /b 1
                    )
                    if not exist "test\\test_sample.py" (
                        echo Test file not found at %CD%\\test\\test_sample.py
                        exit /b 1
                    )

                    if not exist "%REPORT_DIR%" mkdir "%REPORT_DIR%"
                    set "RAW_REPORT=%REPORT_DIR%\\unittest_report.txt"
                    "%PYTHON%" -m pytest "test\\test_sample.py" -v > "%RAW_REPORT%" 2>&1
                    set "TEST_EXIT=%ERRORLEVEL%"

                    if not "%TEST_EXIT%"=="0" (
                        type "%RAW_REPORT%"
                        exit /b %TEST_EXIT%
                    )
                '''
            }
        }

        stage('Publish Report') {
            steps {
                publishHTML(target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: "${env.REPORT_DIR}",
                    reportFiles: "${env.REPORT_FILE}",
                    reportName: 'PyUnit Test Report'
                ])
            }
        }

        stage('Tag And Push') {
            when {
                expression { currentBuild.currentResult == 'SUCCESS' }
            }
            steps {
                script {
                    def ts = new Date().format('yyyyMMdd_HHmmss')
                    def tagName = "Jenkins_${ts}"
                    bat """
                        git config user.name "Jenkins"
                        git config user.email "jenkins@local"
                        git tag ${tagName}
                        set "ENC_PASS=%GIT_PASSWORD:@=%40%"
                        set "REPO_WITH_CREDS=%GIT_REPO_URL:https://=https://%GIT_USERNAME%:%ENC_PASS%@%"
                        git push %REPO_WITH_CREDS% ${tagName}
                    """
                }
            }
        }
    }
}
