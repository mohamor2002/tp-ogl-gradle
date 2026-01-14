pipeline {
    agent any

    stages {
        // 2.1 La phase Test
        stage('Test') {
            steps {
                // 1. Lancement des tests unitaires (Windows)
                bat 'gradlew test'
            }
            post {
                always {
                    // 2. Archivage des résultats des tests unitaires
                    junit 'build/test-results/test/*.xml'

                    // 3. Génération des rapports de tests Cucumber
                    // UPDATED: Looks in 'reports/' to match your Java code
                    cucumber 'reports/*.json'
                }
            }
        }

        // 2.2 La phase Code Analysis
        stage('Code Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    // Analyse la qualité du code avec SonarQube
                    bat 'gradlew sonar'
                }
            }
        }

        // 2.3 La phase Code Quality
        stage('Code Quality') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Stop pipeline if Quality Gate is Failed
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        // 2.4 La phase Build
        stage('Build') {
            steps {
                // 1. Génération du fichier Jar
                bat 'gradlew assemble'

                // 2. Génération de la documentation
                bat 'gradlew javadoc'
            }
            post {
                success {
                    // 3. Archivage du fichier Jar et de la documentation
                    archiveArtifacts artifacts: 'build/libs/*.jar, build/docs/javadoc/**'
                }
            }
        }

        // 2.5 La phase Deploy
        stage('Deploy') {
            steps {
                // Déployer le fichier Jar
                bat 'gradlew publish'
            }
        }
    }

    // 2.6 La phase Notification
    post {
        success {
            // Email Notification success
            mail to: 'lm_amor@esi.dz',
                    subject: "Success: ${currentBuild.fullDisplayName}",
                    body: "The build and deploy were successful."

            // Slack Notification success
            slackSend color: 'good',
                    channel: 'tp_ogl_gradle', // CHANGE THIS to your actual channel name
                    message: "Build Success: ${currentBuild.fullDisplayName} (<${env.BUILD_URL}|Open>)"
        }
        failure {
            // Email Notification failure
            mail to: 'lm_amor@esi.dz',
                    subject: "Failed: ${currentBuild.fullDisplayName}",
                    body: "The pipeline failed in stage: ${env.STAGE_NAME}"

            // Slack Notification failure
            slackSend color: 'danger',
                    channel: 'tp_ogl_gradle', // CHANGE THIS to your actual channel name
                    message: "Build Failed: ${currentBuild.fullDisplayName} in stage ${env.STAGE_NAME} (<${env.BUILD_URL}|Open>)"
        }
    }
}