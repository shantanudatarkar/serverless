pipeline {
    agent any

    tools {
        nodejs "nodejs"
    }

    stages {
        stage('Cleanup') {
            steps {
                echo "Current branch: ${env.BRANCH_NAME}"
                sh 'npm cache clean -f'
            }
        }

        stage('Install') {
            steps {
                echo "Current branch: ${env.BRANCH_NAME}"
                script {
                    def serverlessInstalled = sh(script: 'npm list -g --depth=0 | grep -q serverless', returnStatus: true)
                       sh 'git checkout development'
                    if (serverlessInstalled != 0) {
                        sh 'npm install -g serverless'
                    } else {
                        echo 'serverless is already installed globally'
                    }
                }
            }
        }

        stage('Install Plugin') {
            steps {
                echo "Current branch: ${env.BRANCH_NAME}"
                sh 'npm install -g serverless-offline'
                sh 'npm install -g serverless-plugin-log-retention'
                sh 'npm install -g serverless-stage-manager'
                sh 'npm install -g serverless-enable-api-logs'
            }
        }

        stage('Development') {
           when {
              branch 'development'
            steps 
              script {
                expression { env.BRANCH_NAME == 'development' }           
                echo "Current branch: ${env.BRANCH_NAME}"
                sh 'mvn clean install'
                withCredentials([amazonWebCredentials(credentialsId: 'aws_cred', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY', region: 'ap-south-1')]) {
                sh "serverless deploy --stage development"
                }
            }
        }
    }
        stage('Staging') {
            when {
                expression { env.BRANCH_NAME == 'staging' }
            }
            steps {
                echo "Current branch: ${env.BRANCH_NAME}"
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
                expression { env.BRANCH_NAME == 'master' }
            }
            steps {
                echo "Current branch: ${env.BRANCH_NAME}"
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
