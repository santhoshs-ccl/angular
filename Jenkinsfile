pipeline {
    agent any

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
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
                [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
                nvm install 18
                nvm use 18
                echo "Node version: $(node -v)"
                echo "NPM version: $(npm -v)"
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
                input(
                    message: "⚠️ Approve PRODUCTION deployment from MAIN branch?",
                    ok: "Approve & Deploy",
                    submitter: "admin,qa",
                    submitterParameter: "APPROVED_BY"
                )

                echo "🚀 Deploying PRODUCTION (Approved by: ${env.APPROVED_BY})"
                sh '''
                mkdir -p deploy/prod
                cp -r dist/* deploy/prod/
                '''
            }
        }

    } // stages

    post {
        success {
            echo "✅ Pipeline completed successfully"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}

