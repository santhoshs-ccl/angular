pipeline {
    agent any

    environment {
        NODE_VERSION = '18.16.0'  // Change to your required Node version
        NVM_DIR = "${HOME}/.nvm"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Node & NPM') {
            steps {
                sh '''
                    # Load NVM
                    export NVM_DIR="$HOME/.nvm"
                    [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"

                    # Install and use the required Node version
                    nvm install ${NODE_VERSION}
                    nvm use ${NODE_VERSION}

                    # Show versions
                    node -v
                    npm -v
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build') {
            steps {
                script {
                    // Branch-based build
                    if (env.BRANCH_NAME == 'main') {
                        sh 'ng build --configuration=production'
                    } else if (env.BRANCH_NAME == 'dev') {
                        sh 'ng build --configuration=development'
                    } else if (env.BRANCH_NAME == 'qa') {
                        sh 'ng build --configuration=qa'
                    } else {
                        sh 'ng build'
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Branch-based deployment logic
                    if (env.BRANCH_NAME == 'main') {
                        echo "Deploying to Production..."
                        sh './deploy-prod.sh'
                    } else if (env.BRANCH_NAME == 'dev') {
                        echo "Deploying to Dev..."
                        sh './deploy-dev.sh'
                    } else if (env.BRANCH_NAME == 'qa') {
                        echo "Deploying to QA..."
                        sh './deploy-qa.sh'
                    } else {
                        echo "No deployment for this branch."
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Build & Deployment succeeded for branch ${env.BRANCH_NAME}!"
        }
        failure {
            echo "Build or Deployment failed for branch ${env.BRANCH_NAME}."
        }
    }
}

