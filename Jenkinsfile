pipeline {
    agent any

    options {
        // Keep only last 10 builds
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Checking out source code..."
                checkout scm
            }
        }

        stage('Pre-Build Approval') {
            steps {
                // Developer or Admin can start the build
                input message: "Approve build to start?",
                      ok: "Start Build",
                      submitter: "developer,admin"
            }
        }

        stage('Build') {
            steps {
                sh '''
                set -e

                echo "Loading NVM..."
                export NVM_DIR="$HOME/.nvm"
                [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"

                echo "Checking Node & npm versions..."
                node -v
                npm -v

                echo "Installing dependencies..."
                npm install

                echo "Building project..."
                npm run build
                '''
            }
        }

        stage('Deploy to Dev') {
            steps {
                sh '''
                echo "Deploying to Dev environment..."
                mkdir -p deploy/dev
                cp -r dist/* deploy/dev/
                echo "Dev deployment completed."
                '''
            }
        }

        stage('Deploy to Staging') {
            steps {
                input message: "Approve Staging Deployment?",
                      ok: "Deploy",
                      submitter: "admin"

                sh '''
                echo "Deploying to Staging environment..."
                mkdir -p deploy/staging
                cp -r dist/* deploy/staging/
                echo "Staging deployment completed."
                '''
            }
        }

        stage('Deploy to Production') {
            steps {
                input message: "Approve Production Deployment?",
                      ok: "Deploy",
                      submitter: "admin"

                sh '''
                echo "Deploying to Production environment..."
                mkdir -p deploy/prod
                cp -r dist/* deploy/prod/
                echo "Production deployment completed."
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployment pipeline completed successfully!"
        }
        failure {
            echo "❌ Deployment pipeline failed!"
        }
    }
}

