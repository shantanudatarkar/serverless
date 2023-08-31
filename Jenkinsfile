pipeline {
      agent any
    
    stages {
        stage('Init') {
            steps {
                script {
                    lastCommitInfo = sh(script: "git log -1", returnStdout: true).trim()
                    commitContainsSkip = sh(script: "git log -1 | grep 'skip ci'", returnStatus: true)
                    println "commitContainsSkip value: ${commitContainsSkip}" // Add this line
                    slackMessage = "${env.JOB_NAME} ${env.BRANCH_NAME} received a new commit. :java: \nHere is commmit info: ${lastCommitInfo}\n*Console Output*: <${BUILD_URL}/console | (Open)>"
                    //send slack notification of new commit
                    slack_send(slackMessage)
                    //if commit message contains skip ci
                    if (commitContainsSkip != 0) {
                        skippingText = " Skipping Build for ${env.BRANCH_NAME} branch."
                        env.shouldBuild = false
                        currentBuild.result = 'ABORTED'
                        slack_send(skippingText, "warning")
                        error('BUILD SKIPPED')
                    }
                }
            } //step end
        }
        stage('Install') {
            steps {
                slack_send("npm install serverless")
                sh 'sudo npm install -g serverless'
            }
        }

        stage('Install Plugin') {
            steps {
                slack_send("Installing Plugins")
                sh 'serverless plugin install -n serverless-offline'
                sh 'serverless plugin install -n serverless-plugin-log-retention'
                sh 'serverless plugin install -n serverless-stage-manager'
                sh 'serverless plugin install -n serverless-enable-api-logs'
            }
        }

        stage('Development') {
            when {
                branch 'development'
            } //when
            steps {
                slack_send("Development: Building :coding: ")
                sh 'mvn clean install'
                withAWS(credentials: 'aws-key', region: "ap-south-1") {
                    sh "serverless deploy --stage development"
                    // URL="$(serverless info --verbose --stage development | grep ServiceEndpoint | sed s/ServiceEndpoint\:\ //g)"
                    slack_send("Development: Deployed sucessfully. :heavy_check_mark")
                }
            }
        } //development build

        stage('Staging') {
            when {
                branch 'staging'
            } //when
            steps {
                slack_send("Staging: Building :coding: ")
                sh 'mvn clean install'
                withAWS(credentials: 'aws-key', region: "ap-south-1") {
                    sh "serverless deploy --stage staging"
                    slack_send("Staging: Deployed sucessfully. :heavy_check_mark")
                }
            }
        } //stage build

        stage('Production') {
            when {
                branch 'master'
            } //when
            steps {
                slack_send("Production: Building :coding:")
                sh 'mvn clean install'
                withAWS(credentials: 'aws-key', region: "ap-south-1") {
                    sh "serverless deploy --stage production"
                }
            }
        } //Production build
    } //stages
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
            slack_send("${env.BRANCH_NAME} Something went wrong.Build failed. Check here: Console Output*: <${BUILD_URL}/console | (Open)>", "danger")
        }
    } //post
}

def slack_send(slackMessage, messageColor = "good") {
    slackSend channel: slack_channel, color: messageColor, message: slackMessage, teamDomain: slack_teamDomain, tokenCredentialId: slack_token_cred_id, username: 'Jenkins'
}
