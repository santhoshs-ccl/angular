pipeline {
    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Checking out branch: ${env.BRANCH_NAME}"
                checkout scm
            }
        }

        stage('Install Dependencies & Build') {
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

        stage('Deploy - DEVELOP') {
            when {
                branch 'develop'
            }
            steps {
                echo "🚀 Deploying DEVELOP (No Approval Required)"
                sh '''
                mkdir -p deploy/dev
                cp -r dist/* deploy/dev/
                echo "✅ DEVELOP deployment completed"
                '''
            }
        }

        stage('Deploy - MAIN (Production)') {
            when {
                branch 'main'
            }
            steps {
                input message: "Approve PRODUCTION deployment?",
                      ok: "Deploy",
                      submitter: "admin"

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
            echo "🎉 Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
        cleanup {
            cleanWs()
        }
    }
}

