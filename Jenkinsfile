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
                    if not exist "test\\mip_2d_pack.py" (
                        echo Test file not found at %CD%\\test\\mip_2d_pack.py
                        exit /b 1
                    )

                    if not exist "%REPORT_DIR%" mkdir "%REPORT_DIR%"
                    set "RAW_REPORT=%REPORT_DIR%\\unittest_report.txt"
                    "%PYTHON%" -m unittest test.mip_2d_pack -v > "%RAW_REPORT%" 2>&1
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
                        git push https://%GIT_USERNAME%:%GIT_PASSWORD%@%GIT_REPO_URL:~8% ${tagName}
                    """
                }
            }
        }
    }
}
