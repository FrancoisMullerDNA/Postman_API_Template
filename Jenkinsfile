pipeline {
    agent any
    environment {
        // Slack Credentials for Alerting
        SLACK_CHANNEL = '#axept-local-jenkins' // Your Slack channel
        SLACK_CREDENTIALS = 'slack-token' // The ID of your Slack credentials in Jenkins
        POSTMAN_COLLECTION_URL = 'https://api.postman.com/collections/37020964-afef3c34-8d06-4171-89b0-3d0f5fde361b?access_key=PMAT-01J43XWK5HNE5JBDH9E0F6T8N6'
        POSTMAN_ENVIRONMENT_FILE = 'Reqres.postman_environment.json'
        NEWMAN_DATA_SOURCE = '' // Path to data file if provided
    }

    stages {
        stage('Validation') {
            steps {
                catchError(buildResult: 'FAILURE', stageResult: 'FAILURE') {
                    script {
                        // Documentation: This stage verifies the availability of NPM and Newman,
                        // and lists the branches of the git repository for the current branch.
                        echo 'Running Config Test...'
                        bat '''
                            call npm install -g newman
                            call npm install -g newman-reporter-html
                            call npm install -g newman-reporter-htmlextra
                        '''


                        // Check if Node.js is available
                        def nodeVersion = bat(script: 'node --version', returnStatus: true)
                        if (nodeVersion != 0) {
                            error("Node is not available. Please install Node.js to proceed.")
                        }

                        // Check if npm is available
                        def npmVersion = bat(script: 'npm --version', returnStatus: true)
                        if (npmVersion != 0) {
                            error("npm is not available. Please install Node.js to proceed.")
                        }

                        // Check if newman is available
                        def newmanVersion = bat(script: 'newman --version', returnStatus: true)
                        if (npmVersion != 0) {
                            error("newman is not available. Please install newman to proceed.")
                        }

                        // List branches for the current git repository
                        def gitBranches = bat(script: 'git branch -r', returnStdout: true).trim()
                        echo "Available git branches:\n${gitBranches}"
                    }
                }
            }
        }
        stage('Suite Setup') {
            steps {
                script {
                    echo 'Setting up test suite'
                    // Add any suite setup commands here
                }
            }
        }
        stage('Data Setup') {
            steps {
                script {
                    echo 'Setting up data for tests'
                    // Add any data setup commands here
                }
            }
        }
        stage('Test Setup') {
            steps {
                script {
                        echo 'Setting up tests'

                        // Check if the environment file exists before copying
                        if (!fileExists('C:\\AutomationTools\\Environments\\Reqres.postman_environment.json')) {
                            error("Environment resource not available. Please provide and try again.")
                        }

                        // Copy the environment file
                        bat 'copy C:\\AutomationTools\\Environments\\Reqres.postman_environment.json %WORKSPACE%'

                        // Check if the environment file exists after copying
                        if (!fileExists('Reqres.postman_environment.json')) {
                            error("Failed to copy environment resource to workspace.")
                        }

                        echo 'Environment Collection Successfully copied'
                    }
            }
        }
        stage('Testing') {
            steps {
                catchError(buildResult: 'FAILURE', stageResult: 'FAILURE') {
                    script {
                        // Documentation: This stage runs the Newman tests and generates reports
                        echo 'Running Tests...'

                        // Define the results directory name based on the git branch and build number
                        env.BRANCH_NAME = env.BRANCH_NAME ?: 'default-branch'
                        env.RESULTS_DIR = "Results\\${env.BRANCH_NAME}\\${env.BUILD_NUMBER}"

                        // Create the subdirectories within the Results directory
                        bat "mkdir ${env.RESULTS_DIR}"

                        bat """
                            newman run ${env.POSTMAN_COLLECTION_URL} -e ${env.POSTMAN_ENVIRONMENT_FILE} -r cli,html,junit,htmlextra --reporter-htmlextra-export ${env.RESULTS_DIR}\\HTML_ExtraResults.html --reporter-junit-export ${env.RESULTS_DIR}\\JunitResults.xml --reporter-cli-export ${env.RESULTS_DIR}\\CLIResults.txt --reporter-html-export ${env.RESULTS_DIR}\\HTMLResults.html
                        """
                        }

                        // Store test results in environment variables for later use
                        if (fileExists("${env.RESULTS_DIR}\\JunitResults.xml")) {
                            def report = readFile("${env.RESULTS_DIR}\\JunitResults.xml")
                            // Add logic here to parse the Newman HTML report if needed
                            def testResults = new XmlSlurper().parseText(report)
                            env.TESTS_RAN = testResults.suite.test.size().toString()
                            env.TESTS_PASSED = testResults.suite.test.findAll { it.status.'@status' == 'PASS' }.size().toString()
                            env.TESTS_FAILED = (env.TESTS_RAN.toInteger() - env.TESTS_PASSED.toInteger()).toString()
                        }
                    }
                }
            }
        }
        stage('Reporting') {
            steps {
                catchError(buildResult: 'FAILURE', stageResult: 'FAILURE') {
                    script {
                        // Documentation: Stage to handle report uploads to Confluence and Testmo
                        echo 'Report Uploading reports...'
                    }
                }
            }
        }
        stage('Archive Reports') {
            steps {
                catchError(buildResult: 'FAILURE', stageResult: 'FAILURE') {
                    script {
                        // Documentation: This stage archives the generated reports to Jenkins
                        echo 'Archiving reports...'
                        archiveArtifacts artifacts: "${env.RESULTS_DIR}\\*.html, ${env.RESULTS_DIR}\\*.xml", allowEmptyArchive: true
                    }
                }
            }
        }
        stage('Alerting') {
            steps {
                catchError(buildResult: 'FAILURE', stageResult: 'FAILURE') {
                    script {
                        // Documentation: Section For Slack Notification Integration
                        echo 'Alerting...'
                    }
                }
            }
        }
    }
    post {
        success {
            script {
                // Documentation: This block runs when the pipeline is successful
                currentBuild.result = 'SUCCESS'
            }
        }
        failure {
            script {
                // Documentation: This block runs when the pipeline fails
                currentBuild.result = 'FAILURE'
            }
        }
    }
}
