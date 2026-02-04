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
                powershell '''
                    $ErrorActionPreference = "Stop"
                    $python = Join-Path $env:VENV_PATH "Scripts\\python.exe"
                    if (!(Test-Path $python)) {
                        throw "Python executable not found at $python"
                    }
                    if (!(Test-Path "test\\two_dim_pack_test.py")) {
                        throw "Test file not found at $(Join-Path (Get-Location) 'test\\two_dim_pack_test.py')"
                    }

                    New-Item -ItemType Directory -Path $env:REPORT_DIR -Force | Out-Null
                    $output = & $python -m unittest test.two_dim_pack_test -v 2>&1
                    $exitCode = $LASTEXITCODE
                    $escaped = $output | ForEach-Object { $_ -replace "&", "&amp;" -replace "<", "&lt;" -replace ">", "&gt;" }
                    $html = @"
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>PyUnit Test Report</title>
    <style>
      body { font-family: Arial, sans-serif; margin: 16px; }
      pre { background: #f5f5f5; padding: 12px; border-radius: 6px; }
    </style>
  </head>
  <body>
    <h1>PyUnit Test Report</h1>
    <pre>$($escaped -join "`n")</pre>
  </body>
</html>
"@
                    $reportPath = Join-Path $env:REPORT_DIR $env:REPORT_FILE
                    $html | Set-Content -Path $reportPath -Encoding UTF8

                    if ($exitCode -ne 0) {
                        Write-Host ($output -join "`n")
                        throw "Unit tests failed"
                    }
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
