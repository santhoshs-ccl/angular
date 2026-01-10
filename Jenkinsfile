pipeline {
    agent any

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Branch detected: ${env.BRANCH_NAME}"
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh '''
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
                input message: "⚠️ Approve PRODUCTION deployment from MAIN branch?",
                      ok: "Approve & Deploy",
                      submitter: "admin"

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

