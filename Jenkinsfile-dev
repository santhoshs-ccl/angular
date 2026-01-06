pipeline {
    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Checking out DEV source code..."
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
            steps {
                sh '''
                echo "Deploying to DEV environment..."
                mkdir -p deploy/dev
                cp -r dist/* deploy/dev/
                echo "DEV deployment completed."
                '''
            }
        }
    }

    post {
        success {
            echo "✅ DEV pipeline completed successfully!"
        }
        failure {
            echo "❌ DEV pipeline failed!"
        }
    }
}

