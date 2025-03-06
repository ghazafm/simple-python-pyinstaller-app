pipeline {
    agent none
    environment {
        SLACK_CHANNEL = '#devops'
    }
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'python:3.8-alpine'
                }
            }
            steps {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'safesecurity/pytest'
                }
            }
            steps {
                sh 'pytest --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
        stage('Manual Approval') {
            steps {
                input message: 'Deploy to Production?'
            }
        }
        stage('Deploy') {
            agent {
                docker {
                    image 'python:3.8-alpine'
                    args '--user root'
                }
            }
            steps {
                sh 'pip install --no-cache-dir pytest'
                sleep time: 1, unit: 'MINUTES'
                sh 'pytest --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                success {
                    archiveArtifacts 'dist/add2vals'
                }
            }
        }
    }
    post {
        always {
            script {
                def buildDuration = (currentBuild.duration / 1000).intValue()
                def buildTime = String.format("%02d:%02d", (buildDuration / 60).intValue(), (buildDuration % 60).intValue())

                def color = (currentBuild.currentResult == 'SUCCESS') ? 'good' : 'danger'
                def message = "*${currentBuild.currentResult}* ✅ Job: *${env.JOB_NAME}* Build: *${env.BUILD_NUMBER}*\n" +
                            "Branch: master\n" +
                            "Build Duration: *${buildTime}*\n" +
                            "More info: <${env.BUILD_URL}|View Build>"

                slackSend(channel: SLACK_CHANNEL, color: color, message: message)
            }
        }
    }
}
