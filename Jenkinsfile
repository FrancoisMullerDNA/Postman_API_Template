pipeline {
    agent any
    parameters {
        choice(name: 'TEST_SUITE', choices: ['Smoke', 'Component', 'Integration', 'Functional', 'Performance'], description: 'Select the Test Suite to run')
    }
    environment {
        SLACK_CHANNEL = '#axept-local-jenkins' // Your Slack channel
        SLACK_CREDENTIALS = 'slack-token' // The ID of your Slack credentials in Jenkins
        POSTMAN_ENVIRONMENT_FILE = 'Orical.postman_environment.json'
        NEWMAN_DATA_SOURCE = '' // Path to data file if provided
    }

    stages {
        stage('Preparing Tokens') {
            steps {
                script {
                    // Use the withCredentials block to pull the secret and set it to POSTMAN_PAT_KEY
                    withCredentials([string(credentialsId: 'postman-access-token', variable: 'GITHUB_TOKEN')]) {
                        env.POSTMAN_PAT_KEY = GITHUB_TOKEN
                    }
                    env.POSTMAN_COLLECTION_URL = "https://api.postman.com/collections/37020964-bff97c6e-2378-4e6a-ba32-29a17c5f7b19?access_key=${env.POSTMAN_PAT_KEY}"
                }
            }
        }
        stage('Validation') {
            steps {
                catchError(buildResult: 'FAILURE', stageResult: 'FAILURE') {
                    script {
                        // Documentation: This stage verifies the availability of NPM and Newman,
                        // and lists the branches of the git repository for the current branch.
                        echo 'Running Config Test...'

                        env.BRANCH_NAME = env.BRANCH_NAME ?: 'default-branch'
                        env.RESULTS_DIR = "Results\\${env.BRANCH_NAME}\\${env.BUILD_NUMBER}\\${params.TEST_SUITE}"


                        def testSuite = (params.TEST_SUITE?.trim()) ? params.TEST_SUITE : 'Smoke'
                        currentBuild.description = "Active Test Suite: ${testSuite}"
                        echo "${params.TEST_SUITE}"

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
                        if (newmanVersion != 0) {
                            error("Newman is not available. Please install Newman to proceed.")
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
                    if (!fileExists('C:\\AutomationTools\\Environments\\Orical.postman_environment.json')) {
                        error("Environment resource not available. Please provide and try again.")
                    }

                    // Copy the environment file
                    bat 'copy "C:\\AutomationTools\\Environments\\Orical.postman_environment.json" %WORKSPACE%'

                    // Check if the environment file exists after copying
                    if (!fileExists('C:\\AutomationTools\\Environments\\Orical.postman_environment.json')) {
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
                        env.RESULTS_DIR = "Results\\${env.BRANCH_NAME}\\${env.BUILD_NUMBER}\\${params.TEST_SUITE}"

                        // Create the subdirectories within the Results directory
                        bat "mkdir ${env.RESULTS_DIR}"

                        // Run the Newman tests
                        if (env.NEWMAN_DATA_SOURCE) {
                            bat """
                                newman run ${env.POSTMAN_COLLECTION_URL} -e ${env.POSTMAN_ENVIRONMENT_FILE} --env-var TestSuite=${params.TEST_SUITE} --iteration-data ${env.NEWMAN_DATA_SOURCE} -r cli,html,junit,htmlextra \
                                --reporter-htmlextra-export ${env.RESULTS_DIR}\\HTML_ExtraResults.html \
                                --reporter-junit-export ${env.RESULTS_DIR}\\JunitResults.xml \
                                --reporter-cli-export ${env.RESULTS_DIR}\\CLIResults.txt \
                                --reporter-html-export ${env.RESULTS_DIR}\\HTMLResults.html
                            """
                        } else {
                            bat """
                                newman run ${env.POSTMAN_COLLECTION_URL} -e ${env.POSTMAN_ENVIRONMENT_FILE} --env-var TestSuite=${params.TEST_SUITE} -r cli,html,junit,htmlextra \
                                --reporter-htmlextra-export ${env.RESULTS_DIR}\\HTML_ExtraResults.html \
                                --reporter-junit-export ${env.RESULTS_DIR}\\JunitResults.xml \
                                --reporter-cli-export ${env.RESULTS_DIR}\\CLIResults.txt \
                                --reporter-html-export ${env.RESULTS_DIR}\\HTMLResults.html
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
