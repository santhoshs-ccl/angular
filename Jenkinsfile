pipeline {
    agent any

    environment {
        NODE_VERSION = '24.12.0'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    // Detect the branch name properly
                    BRANCH_NAME = sh(
                        script: 'git rev-parse --abbrev-ref HEAD',
                        returnStdout: true
                    ).trim()
                    echo "Detected branch: ${BRANCH_NAME}"
                }
            }
        }

        stage('Setup Node') {
            steps {
                script {
                    // Load NVM and use the correct Node version
                    sh '''
                        export NVM_DIR="$HOME/.nvm"
                        [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
                        nvm install ${NODE_VERSION}
                        nvm use ${NODE_VERSION}
                        node -v
                        npm -v
                    '''
                }
            }
        }

        stage('Build & Deploy') {
            steps {
                script {
                    if (BRANCH_NAME == 'main') {
                        echo "Building and deploying to Production"
                        sh 'npm install && npm run build:prod'
                        // Add deploy commands here
                    } else if (BRANCH_NAME == 'dev') {
                        echo "Building and deploying to Development"
                        sh 'npm install && npm run build:dev'
                        // Add deploy commands here
                    } else if (BRANCH_NAME == 'qa') {
                        echo "Building and deploying to QA"
                        sh 'npm install && npm run build:qa'
                        // Add deploy commands here
                    } else {
                        echo "Branch not recognized, skipping deploy"
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Cleaning workspace"
            cleanWs()
        }
    }
}

