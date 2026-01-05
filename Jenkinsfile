pipeline {
    agent any

    environment {
        // Ensure Node & npm are available on macOS
        PATH = "/opt/homebrew/bin:/usr/local/bin:${env.PATH}"
    }

    options {
        // Keep only 10 builds to save disk space
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
                // Pause build until Admin/QA approve
                input message: "Approve build to start?", ok: "Start Build", submitter: "qa,admin"
            }
        }

        stage('Build') {
            steps {
                sh '''
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
                echo "Ready for Staging deployment..."
                input message: "Approve Staging Deployment?", ok: "Deploy", submitter: "qa,admin"
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
                echo "Ready for Production deployment..."
                input message: "Approve Production Deployment?", ok: "Deploy", submitter: "admin"
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

