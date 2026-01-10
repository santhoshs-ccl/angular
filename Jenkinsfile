pipeline {
    agent any

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()
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
                npm install
                npm run build
                '''
            }
        }

        stage('Deploy - DEVELOP') {
            when {
                branch 'develop'
            }
            steps {
                echo "🚀 Deploying DEVELOP (No approval required)"
                sh '''
                mkdir -p deploy/dev
                cp -r dist/* deploy/dev/
                echo "✅ DEVELOP deployment completed"
                '''
            }
        }

        stage('Deploy - MAIN (PRODUCTION)') {
            when {
                beforeInput true
                branch 'main'
            }
            input {
                message "⚠️ Approve PRODUCTION deployment from MAIN branch?"
                ok "Approve & Deploy"
                submitter "admin"
            }
            steps {
                echo "🚀 Deploying PRODUCTION"
                sh '''
                mkdir -p deploy/prod
                cp -r dist/* deploy/prod/
                echo "✅ PRODUCTION deployment completed"
                '''
            }
        }
    }

    post {
        success {
            echo "🎉 Pipeline completed successfully"
        }
        failure {
            echo "❌ Pipeline failed"
        }
        cleanup {
            cleanWs()
        }
    }
}

