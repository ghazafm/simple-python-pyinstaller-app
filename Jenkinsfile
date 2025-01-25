def COLOR_MAP = [
    'SUCCESS': 'good',  // Green for successful builds
    'FAILURE': 'danger' // Red for failed builds
]

node {
    stage('Checkout') {
        checkout scm
    }

    stage('Build') {
        docker.image('python:2-alpine').inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
        }
    }
    stage('Test') {
        docker.image('qnib/pytest').inside {
            sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
        }
        junit 'test-reports/results.xml'
    }
    stage('Deliver') {
        def success = false

        try {
            docker.image('cdrx/pyinstaller-linux:python2').inside("-v ${pwd()}:/workspace --timeout=600") {
                sh 'pyinstaller --onefile sources/add2vals.py'
            }
            deliverSuccess = true
        }catch (Exception e) {
            currentBuild.result = 'FAILURE'
            throw e
        } finally {
            if (deliverSuccess) {
                archiveArtifacts 'dist/add2vals'
            }
        }
    }

    // Saya mencoba menambahkan fitur notifikasi yang dikirim ke channel slack saya
    post {
        always {
            script {
                def buildDuration = (currentBuild.duration / 1000).intValue()
                def buildTime = String.format("%02d:%02d", (buildDuration / 60).intValue(), (buildDuration % 60).intValue())

                def message = [
                    "*${currentBuild.currentResult}* :white_check_mark: Job: *${env.JOB_NAME}* Build: *${env.BUILD_NUMBER}*",
                    "Branch: *${GIT_BRANCH}*",
                    "Build Duration: *${buildTime}*",
                    "More info: <${env.BUILD_URL}|View Build>"
                ].join("\n")

                if (currentBuild.currentResult == 'SUCCESS') {
                    message += "\nArtifact: *vproapp-${env.BUILD_ID}-${env.BUILD_TIMESTAMP}.war* Delivered."
                } else {
                    message += "\n:exclamation: *Check logs for errors.* Contact the DevOps team if assistance is needed."
                }

                slackSend(
                    channel: '#devops',
                    color: COLOR_MAP[currentBuild.currentResult],
                    message: message
                )
            }
        }
    }
}