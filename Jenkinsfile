pipeline {
    agent any

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        // Use NVM directory
        NVM_DIR = "${HOME}/.nvm"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "📌 Branch detected: ${env.BRANCH_NAME}"
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh '''
                set -e
                # Load NVM
                [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"

                # Use supported Node version
                nvm install 18
                nvm use 18

                echo "Node version: $(node -v)"
                echo "NPM version: $(npm -v)"

                # Install dependencies and build
                npm install
                npm run build
                '''
            }
        }

        stage('Deploy - DEVELOP') {
            when {
                expression { env.BRANCH_NAME == 'develop' }
            }
            steps {
                echo "🚀 Deploying DEVELOP (No approval)"
                sh '''
                mkdir -p deploy/dev
                cp -r dist/* deploy/dev/
                '''
            }
        }

        stage('Deploy - MAIN (PRODUCTION)') {
            when {
                expression { env.BRANCH_NAME == 'main' }
            }
            steps {
                // Input approval for main
                input message: "⚠️ Approve PRODUCTION deployment from MAIN branch?",
                      ok: "Approve & Deploy",
#                      submitter: "qa,admin"   // users allowed to approve

                echo "🚀 Deploying PRODUCTION"
                sh '''
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

