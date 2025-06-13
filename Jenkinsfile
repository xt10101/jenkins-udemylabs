pipeline {
    agent any
    tools {
        go 'go-1.19' // Comes from the jenkins global config
    }
    environment {
        ENV = "${env.BRANCH_NAME == 'master' ? 'PROD' : 'DEV'}" // Define the ENV based on the branch name
        BRANCH = "${env.BRANCH_NAME}"
    }

    stages {
        stage('Send initial notification') {
            when {
                anyOf {
                    branch 'master';
                    branch 'develop'
                }
            }
            steps {
                slackSend channel: '#general',
                          message: "Build for job ${env.JOB_NAME} has started - (<${env.BUILD_URL}|Open>)"
            }
        }
        stage('Build') {
            steps {
                sh 'bash scripts/build.sh' // Run the build
            }
        }
        stage('Test') {
            steps {
                sh 'bash scripts/test.sh' // Run the test
            }
        }
        stage('Deploy') {
            when {
                anyOf {
                    branch 'master';
                    branch 'develop'
                }
            }
            steps {
                sh 'export JENKINS_NODE_COOKIE=do_not_kill ; bash scripts/deploy.sh'
            }
        }
    }
    post {
        always {
            slackSend channel: '#general',
                      color: "${currentBuild.currentResult == 'SUCCESS' ? 'good' : 'danger'}",
                      message: "Build for job ${env.JOB_NAME} finished with status ${currentBuild.currentResult} - (<${env.BUILD_URL}|Open>)"
        }
    }
}

