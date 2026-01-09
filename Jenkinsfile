pipeline {
    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        // Ensure BRANCH_NAME is set, fallback to git detection if null
        BRANCH_NAME = "${env.BRANCH_NAME ?: sh(script: 'git rev-parse --abbrev-ref HEAD', returnStdout: true).trim()}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Checking out source code from ${env.BRANCH_NAME}"
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh '''
                set -e
                export NVM_DIR="$HOME/.nvm"
                [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"

                node -v
                npm -v
                npm install
                npm run build
                '''
            }
        }

        stage('Deploy to DEV') {
            when {
                expression { env.BRANCH_NAME == 'develop' }
            }
            steps {
                sh '''
                echo "Auto deploying to DEV..."
                mkdir -p deploy/dev
                cp -r dist/* deploy/dev/
                '''
            }
        }

        stage('Admin Approval for PROD') {
            when {
                expression { env.BRANCH_NAME == 'main' }
            }
            steps {
                input message: "Approve PRODUCTION deployment?",
                      ok: "Approve",
                      submitter: "admin"
            }
        }

        stage('Deploy to PRODUCTION') {
            when {
                expression { env.BRANCH_NAME == 'main' }
            }
            steps {
                sh '''
                echo "Deploying to PRODUCTION..."
                mkdir -p deploy/prod
                cp -r dist/* deploy/prod/
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}

