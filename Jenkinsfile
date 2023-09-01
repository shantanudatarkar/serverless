pipeline {
    agent any

    environment {
        BRANCH_NAME = 'development'
    }

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

        stage('Remove node_modules') {
            steps {
                sh 'rm -rf node_modules'
            }
        }

        stage('Install AWS SDK') {
            steps {
                sh 'npm install aws-sdk'
            }
        }

        stage('Install Dependencies') {
            steps {
                echo "Current branch: ${env.BRANCH_NAME}"
                sh 'npm install'
            }
        }

        stage('Development') {
            when {
                expression { BRANCH_NAME == 'development' }
            }
            steps {
                script {
                    echo "Current branch: ${BRANCH_NAME}"
                    sh 'mvn clean install'
                     sh 'npm install -g aws-sdk'
                    sh 'serverless --clear-caches'
                    sh "serverless deploy --stage development"

                    withCredentials([amazonWebCredentials(credentialsId: 'aws_cred', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY', region: 'ap-south-1')]) {
                        def deployResult = sh(script: "serverless deploy --stage development", returnStatus: true)
                        if (deployResult == 0) {
                            currentBuild.result = 'SUCCESS'
                        } else {
                            currentBuild.result = 'FAILURE'
                        }
                    }
                }
            }
        }

        stage('Staging') {
            when {
                expression { BRANCH_NAME == 'staging' }
            }
            steps {
                echo "Current branch: ${BRANCH_NAME}"
                slack_send("Staging: Building :coding: ")
                sh 'mvn clean install'
                withAWS(credentials: 'aws-key', region: "ap-south-1") {
                    sh "serverless deploy --stage staging"
                    //slack_send("Staging: Deployed successfully. :heavy_check_mark")
                }
            }
        }

        stage('Production') {
            when {
                expression { BRANCH_NAME == 'master' }
            }
            steps {
                echo "Current branch: ${BRANCH_NAME}"
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
            slack_send("${BRANCH_NAME} Build Completed Successfully. Check here: Console Output*: <${BUILD_URL}/console | (Open)>", "#0066FF")
        }
        aborted {
            slack_send("Jenkins build Skipped/Aborted.", "warning")
        }
        failure {
            slack_send("${BRANCH_NAME} Something went wrong. Build failed. Check here: Console Output*: <${BUILD_URL}/console | (Open)>", "danger")
        }
    }
}

def slack_send(slackMessage, messageColor = "good") {
    slackSend channel: slack_channel, color: messageColor, message: slackMessage, teamDomain: slack_teamDomain, tokenCredentialId: slack_token_cred_id, username: 'Jenkins'
}
