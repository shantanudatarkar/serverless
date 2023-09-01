pipeline {
    agent any

    tools {
        // Use the name you provided in the Global Tool Configuration
        nodejs "nodejs"
    }

    stages {
        stage('Install') {
            steps {
                //slack_send("npm install serverless")
                sh 'npm install -g serverless'
            }
        }
        stage('Install Plugin') {
            steps {
                //slack_send("Installing Plugins")
                sh 'npm install -g serverless-offline'
                sh 'npm install -g serverless-plugin-log-retention' // Corrected plugin name
                sh 'npm install -g serverless-stage-manager'
                sh 'npm install -g serverless-enable-api-logs'
            }
        }
        stage('Development') {
            when {
                branch 'development'
            }
            steps {
                slack_send("Development: Building :coding: ")
                sh 'mvn clean install'
                withAWS(credentials: 'aws-key', region: "ap-south-1") {
                    sh "serverless deploy --stage development"
                    slack_send("Development: Deployed successfully. :heavy_check_mark")
                }
            }
        }
        stage('Staging') {
            when {
                branch 'staging'
            }
            steps {
                slack_send("Staging: Building :coding: ")
                sh 'mvn clean install'
                withAWS(credentials: 'aws-key', region: "ap-south-1") {
                    sh "serverless deploy --stage staging"
                    slack_send("Staging: Deployed successfully. :heavy_check_mark")
                }
            }
        }
        stage('Production') {
            when {
                branch 'master'
            }
            steps {
                slack_send("Production: Building :coding:")
                sh 'mvn clean install'
                withAWS(credentials: 'aws-key', region: "ap-south-1") {
                    sh "serverless deploy --stage production"
                }
            }
        }
    }

    post {
        always {
            sh "chmod -R 777 ."
            deleteDir()
        }
        success {
            slack_send("${env.BRANCH_NAME} Build Completed Successfully. Check here: Console Output*: <${BUILD_URL}/console | (Open)>", "#0066FF")
        }
        aborted {
            slack_send("Jenkins build Skipped/Aborted.", "warning")
        }
        failure {
            slack_send("${env.BRANCH_NAME} Something went wrong. Build failed. Check here: Console Output*: <${BUILD_URL}/console | (Open)>", "danger")
        }
    }
}

def slack_send(slackMessage, messageColor = "good") {
    slackSend channel: slack_channel, color: messageColor, message: slackMessage, teamDomain: slack_teamDomain, tokenCredentialId: slack_token_cred_id, username: 'Jenkins'
}
