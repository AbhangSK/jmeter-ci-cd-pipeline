pipeline {
    agent any

    environment {
        JMETER_HOME = "C:\\JMeterInstallation\\apache-jmeter-5.6.3"
        TEST_PLAN = "Altoro Mutual Home Page.jmx"
        RESULT_FILE = "results\\result.jtl"
        REPORT_DIR = "reports"
        ERROR_THRESHOLD = 2
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/AbhangSK/jmeter-ci-cd-pipeline.git'
            }
        }

        stage('Run JMeter Test') {
            steps {
                bat """
                if not exist results mkdir results
                if not exist reports mkdir reports

                %JMETER_HOME%\\bin\\jmeter.bat ^
                -n ^
                -t %TEST_PLAN% ^
                -l %RESULT_FILE% ^
                -e ^
                -o %REPORT_DIR%
                """
            }
        }

        stage('Performance Gate') {
            steps {
                script {

                    def total = 0
                    def failed = 0

                    def lines = readFile(env.RESULT_FILE).split("\\r?\\n")

                    for (line in lines.drop(1)) {

                        if (line.trim()) {

                            total++

                            def cols = line.split(',')

                            if (cols.size() > 7) {

                                def success = cols[7]

                                if (success == "false") {
                                    failed++
                                }
                            }
                        }
                    }

                    def errorPercent = (failed * 100.0) / total

                    echo "Total Samples: ${total}"
                    echo "Failed Samples: ${failed}"
                    echo "Error Percentage: ${errorPercent}%"

                    if (errorPercent > env.ERROR_THRESHOLD.toInteger()) {

                        currentBuild.result = 'FAILURE'

                        error("Performance Gate Failed! Error % exceeded threshold.")
                    }
                }
            }
        }

        stage('Publish HTML Report') {
            steps {

                publishHTML(target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'reports',
                    reportFiles: 'index.html',
                    reportName: 'JMeter HTML Report'
                ])
            }
        }
    }

    post {

        always {

            archiveArtifacts artifacts: 'results/*.jtl', fingerprint: true

            archiveArtifacts artifacts: 'reports/**/*', fingerprint: true
        }

        success {
            echo 'Performance Test Passed'
        }

        failure {
            echo 'Performance Test Failed'
        }
    }
}
