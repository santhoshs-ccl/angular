pipeline {
    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        skipDefaultCheckout true
    }

    stages {

        stage('Checkout') {
            steps {
                script {
                    echo "Checking out source code..."
                    checkout scm
                    BRANCH_NAME = env.GIT_BRANCH ?: sh(script: "git rev-parse --abbrev-ref HEAD", returnStdout: true).trim()
                    echo "Current branch: ${BRANCH_NAME}"
                }
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

        stage('Deploy') {
            steps {
                script {
                    if (BRANCH_NAME == 'dev') {
                        echo "Deploying to DEV..."
                        sh '''
                        mkdir -p deploy/dev
                        cp -r dist/* deploy/dev/
                        echo "DEV deployment completed."
                        '''
                    } else if (BRANCH_NAME == 'staging') {
                        input message: "Approve STAGING deployment?", ok: "Deploy", submitter: "team-lead"
                        echo "Deploying to STAGING..."
                        sh '''
                        mkdir -p deploy/staging
                        cp -r dist/* deploy/staging/
                        echo "STAGING deployment completed."
                        '''
                    } else if (BRANCH_NAME == 'main' || BRANCH_NAME == 'prod') {
                        input message: "Final approval for PRODUCTION deployment?", ok: "Deploy", submitter: "admin"
                        echo "Deploying to PRODUCTION..."
                        sh '''
                        mkdir -p deploy/prod
                        cp -r dist/* deploy/prod/
                        echo "PRODUCTION deployment completed."
                        '''
                    } else {
                        echo "Branch ${BRANCH_NAME} is not configured for deployment."
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully for branch: ${BRANCH_NAME}"
        }
        failure {
            echo "❌ Pipeline failed for branch: ${BRANCH_NAME}"
        }
    }
}

