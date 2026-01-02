pipeline {
    agent any

    environment {
        DEPLOY_DIR = "/Users/naveen/deployed-angular-app/angular"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git url: 'file:///Users/naveen/projects/my-angular-app', branch: 'main'
            }
        }

        stage('Install Dependencies') {
            steps {
                echo "Installing npm dependencies..."
                sh 'npm install'
            }
        }

        stage('Build Angular App') {
            steps {
                echo "Building Angular app..."
                sh 'npm run build'   // default output goes to dist/
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying to $DEPLOY_DIR ..."
                sh """
                rm -rf $DEPLOY_DIR/*
                cp -r dist/* $DEPLOY_DIR/
                """
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful!"
        }
        failure {
            echo "❌ Deployment failed!"
        }
    }
}

