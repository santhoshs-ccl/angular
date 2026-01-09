pipeline {
    agent any

    options { 
        buildDiscarder(logRotator(numToKeepStr: '10'))
        skipDefaultCheckout true
    }

    environment {
        NVM_DIR = "$HOME/.nvm"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                script {
                    // Detect branch: use env.BRANCH_NAME if multibranch, else fallback to git command
                    def branchFromGit = sh(script: "git rev-parse --abbrev-ref HEAD", returnStdout: true).trim()
                    env.DEPLOY_BRANCH = env.BRANCH_NAME ?: branchFromGit
                    echo "Detected branch: ${env.DEPLOY_BRANCH}"
                }
            }
        }

        stage('Build') {
            steps {
                sh '''
                set -e
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
                    if (env.DEPLOY_BRANCH == 'dev') {
                        echo "Deploying to DEV..."
                        sh '''
                        mkdir -p deploy/dev
                        cp -r dist/* deploy/dev/
                        echo "DEV deployment completed."
                        '''
                    } else if (env.DEPLOY_BRANCH == 'staging') {
                        input message: "Approve STAGING deployment?", ok: "Deploy", submitter: "team-lead"
                        echo "Deploying to STAGING..."
                        sh '''
                        mkdir -p deploy/staging
                        cp -r dist/* deploy/staging/
                        echo "STAGING deployment completed."
                        '''
                    } else if (env.DEPLOY_BRANCH == 'main' || env.DEPLOY_BRANCH == 'prod') {
                        input message: "Final approval for PRODUCTION deployment?", ok: "Deploy", submitter: "admin"
                        echo "Deploying to PRODUCTION..."
                        sh '''
                        mkdir -p deploy/prod
                        cp -r dist/* deploy/prod/
                        echo "PRODUCTION deployment completed."
                        '''
                    } else {
                        echo "Branch ${env.DEPLOY_BRANCH} is not configured for deployment. Skipping deployment."
                    }
                }
            }
        }
    }

    post {
        success { 
            echo "✅ Pipeline completed successfully for branch: ${env.DEPLOY_BRANCH}" 
        }
        failure { 
            echo "❌ Pipeline failed for branch: ${env.DEPLOY_BRANCH}" 
        }
    }
}

