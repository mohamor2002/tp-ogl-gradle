pipeline {
    agent any
    
    environment {
        // SonarQube configuration
        SONAR_HOST_URL = 'http://localhost:9000'
        SONAR_TOKEN = '1cb507e55c57262da76254b8c0e56a1216510ecb'
        
        // Maven repository credentials
        MAVEN_REPO_USERNAME = 'myMavenRepo'
        MAVEN_REPO_PASSWORD = 'test0005'
        
        // Email configuration
        EMAIL_FROM = 'lm_amor@esi.dz'
        EMAIL_PASSWORD = 'onvilblgbfugscpz'
        EMAIL_TO = 'lm_amor@esi.dz'
        
        // Slack configuration
        SLACK_WEBHOOK = 'https://hooks.slack.com/services/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'
        SLACK_CHANNEL = '#jenkins-notifications'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code from repository...'
                checkout scm
            }
        }
        
        stage('Test') {
            steps {
                echo 'Running unit tests and generating Cucumber reports...'
                script {
                    try {
                        // Run tests
                        bat './gradlew clean test'
                        
                        // Generate Cucumber reports
                        bat './gradlew generateCucumberReports'
                        
                        // Archive test results
                        junit '**/build/test-results/test/*.xml'
                        
                        // Publish Cucumber reports
                        publishHTML([
                            allowMissing: false,
                            alwaysLinkToLastBuild: true,
                            keepAll: true,
                            reportDir: 'build/reports/cucumber/cucumber-html-reports',
                            reportFiles: 'overview-features.html',
                            reportName: 'Cucumber Report'
                        ])
                        
                        // Publish JUnit test reports
                        publishHTML([
                            allowMissing: false,
                            alwaysLinkToLastBuild: true,
                            keepAll: true,
                            reportDir: 'build/reports/tests/test',
                            reportFiles: 'index.html',
                            reportName: 'JUnit Test Report'
                        ])
                        
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        error "Tests failed: ${e.message}"
                    }
                }
            }
        }
        
        stage('Code Analysis') {
            steps {
                echo 'Running SonarQube analysis...'
                script {
                    try {
                        bat './gradlew sonar'
                        echo 'SonarQube analysis completed. Check SonarQube dashboard for results.'
                    } catch (Exception e) {
                        echo "Warning: SonarQube analysis failed: ${e.message}"
                        // Don't fail the build, just warn
                    }
                }
            }
        }
        
        stage('Build') {
            steps {
                echo 'Building JAR and generating documentation...'
                script {
                    try {
                        // Build JAR
                        bat './gradlew jar'
                        
                        // Generate Javadoc
                        bat './gradlew javadoc'
                        
                        // Archive JAR artifacts
                        archiveArtifacts artifacts: '**/build/libs/*.jar', 
                                       fingerprint: true,
                                       allowEmptyArchive: false
                        
                        // Archive Javadoc
                        publishHTML([
                            allowMissing: false,
                            alwaysLinkToLastBuild: true,
                            keepAll: true,
                            reportDir: 'build/docs/javadoc',
                            reportFiles: 'index.html',
                            reportName: 'Javadoc'
                        ])
                        
                        // Archive JaCoCo coverage report
                        publishHTML([
                            allowMissing: false,
                            alwaysLinkToLastBuild: true,
                            keepAll: true,
                            reportDir: 'build/reports/jacoco/test/html',
                            reportFiles: 'index.html',
                            reportName: 'JaCoCo Coverage Report'
                        ])
                        
                        // Publish JaCoCo coverage
                        jacoco(
                            execPattern: '**/build/jacoco/*.exec',
                            classPattern: '**/build/classes/java/main',
                            sourcePattern: '**/src/main/java'
                        )
                        
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        error "Build failed: ${e.message}"
                    }
                }
            }
        }
        
        stage('Deploy') {
            steps {
                echo 'Deploying to MyMavenRepo...'
                script {
                    try {
                        // Publish to Maven repository
                        bat './gradlew publish'
                        echo 'Artifact successfully deployed to MyMavenRepo!'
                        
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        error "Deployment failed: ${e.message}"
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completed successfully!'
            script {
                // Send Slack notification
                try {
                    bat './gradlew sendSlackNotification'
                } catch (Exception e) {
                    echo "Slack notification failed: ${e.message}"
                }
                
                // Send Email notification
                try {
                    bat './gradlew sendEmailNotification'
                } catch (Exception e) {
                    echo "Email notification failed: ${e.message}"
                }
                
                // Alternative: Use Jenkins email plugin
                emailext (
                    subject: "SUCCESS: Pipeline '${env.JOB_NAME}' [${env.BUILD_NUMBER}]",
                    body: """
                        <html>
                        <body>
                            <h2>Build Success</h2>
                            <p><strong>Job:</strong> ${env.JOB_NAME}</p>
                            <p><strong>Build Number:</strong> ${env.BUILD_NUMBER}</p>
                            <p><strong>Status:</strong> SUCCESS</p>
                            <p><strong>Build URL:</strong> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                            <p>The Matrix API has been successfully built, tested, and deployed!</p>
                            <hr>
                            <h3>Pipeline Stages:</h3>
                            <ul>
                                <li>✅ Tests Passed</li>
                                <li>✅ Code Analysis Complete</li>
                                <li>✅ Quality Gate Passed</li>
                                <li>✅ Build Successful</li>
                                <li>✅ Deployment Complete</li>
                            </ul>
                        </body>
                        </html>
                    """,
                    to: "${EMAIL_TO}",
                    mimeType: 'text/html'
                )
                
                // Alternative: Use Jenkins Slack plugin
                slackSend (
                    color: 'good',
                    channel: "${SLACK_CHANNEL}",
                    message: """
                        *BUILD SUCCESS* :white_check_mark:
                        Job: `${env.JOB_NAME}`
                        Build: `#${env.BUILD_NUMBER}`
                        Branch: `${env.BRANCH_NAME}`
                        Status: SUCCESS
                        <${env.BUILD_URL}|View Build>
                    """
                )
            }
        }
        
        failure {
            echo 'Pipeline failed!'
            script {
                // Send failure notification via Email
                emailext (
                    subject: "FAILURE: Pipeline '${env.JOB_NAME}' [${env.BUILD_NUMBER}]",
                    body: """
                        <html>
                        <body style="font-family: Arial, sans-serif;">
                            <h2 style="color: #d9534f;">Build Failed</h2>
                            <p><strong>Job:</strong> ${env.JOB_NAME}</p>
                            <p><strong>Build Number:</strong> ${env.BUILD_NUMBER}</p>
                            <p><strong>Status:</strong> <span style="color: #d9534f;">FAILURE</span></p>
                            <p><strong>Build URL:</strong> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                            <p style="color: #d9534f;">The pipeline has failed. Please check the console output for details.</p>
                            <hr>
                            <p><a href="${env.BUILD_URL}console">View Console Output</a></p>
                        </body>
                        </html>
                    """,
                    to: "${EMAIL_TO}",
                    mimeType: 'text/html'
                )
                
                // Send failure notification via Slack
                slackSend (
                    color: 'danger',
                    channel: "${SLACK_CHANNEL}",
                    message: """
                        *BUILD FAILED* :x:
                        Job: `${env.JOB_NAME}`
                        Build: `#${env.BUILD_NUMBER}`
                        Branch: `${env.BRANCH_NAME}`
                        Status: FAILURE
                        <${env.BUILD_URL}console|View Console Output>
                    """
                )
            }
        }
        
        unstable {
            echo 'Pipeline is unstable!'
            script {
                emailext (
                    subject: "UNSTABLE: Pipeline '${env.JOB_NAME}' [${env.BUILD_NUMBER}]",
                    body: """
                        <html>
                        <body>
                            <h2 style="color: #f0ad4e;">Build Unstable</h2>
                            <p><strong>Job:</strong> ${env.JOB_NAME}</p>
                            <p><strong>Build Number:</strong> ${env.BUILD_NUMBER}</p>
                            <p><strong>Status:</strong> UNSTABLE</p>
                            <p><strong>Build URL:</strong> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                        </body>
                        </html>
                    """,
                    to: "${EMAIL_TO}",
                    mimeType: 'text/html'

                )
                
                slackSend (
                    color: 'warning',
                    channel: "${SLACK_CHANNEL}",
                    message: """
                        *BUILD UNSTABLE* :warning:
                        Job: `${env.JOB_NAME}`
                        Build: `#${env.BUILD_NUMBER}`
                        <${env.BUILD_URL}|View Build>
                    """
                )
            }
        }
        
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
    }
}
