pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                echo "Installing dependencies..."
                sh 'npm install'
                echo "Building project..."
                sh 'npm run build'
            }
        }
        stage('Deploy (Local Test)') {
            steps {
                echo "Running local deploy simulation..."
                sh 'mkdir -p ./local_deploy && cp -r ./dist/* ./local_deploy/'
                echo "Files deployed to ./local_deploy folder"
            }
        }
    }
}

