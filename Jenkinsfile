pipeline {
    agent any

    environment {
        // Make Node & npm visible to Jenkins (macOS fix)
        PATH = "/opt/homebrew/bin:/usr/local/bin:${env.PATH}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh '''
                    echo "Checking Node & npm..."
                    node -v
                    npm -v

                    echo "Installing dependencies..."
                    npm install

                    echo "Building project..."
                    npm run build
                '''
            }
        }

        stage('Deploy (Local Test)') {
            steps {
                sh '''
                    echo "Running local deploy simulation..."
                    mkdir -p local_deploy
                    cp -r dist/* local_deploy/
                    echo "Files deployed to ./local_deploy folder"
                '''
            }
        }
    }
}

