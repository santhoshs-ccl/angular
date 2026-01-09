kkpipeline {
    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
    }

    environment {
        NVM_DIR = "${env.HOME}/.nvm"
    }

    stages {

        stage('Checkout') {
            steps {
                script {
                    echo "Checking out branch: ${env.BRANCH_NAME}"
                    checkout scm
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
                    def branch = env.BRANCH_NAME ?: 'dev'

                    if (branch == 'dev') {
                        echo "Deploying to DEV environment..."
                        sh '''
                            mkdir -p deploy/dev
                            cp -r dist/* deploy/dev/
                            echo "DEV deployment completed."
                        '''
                    }
                    else if (branch == 'qa') {
                        input message: "Approve QA deployment?", ok: "Deploy", submitter: "admin"
                        echo "Deploying to QA environment..."
                        sh '''
                            mkdir -p deploy/qa
                            cp -r dist/* deploy/qa/
                            echo "QA deployment completed."
                        '''
                    }
                    else if (branch == 'main') {
                        input message: "Approve PRODUCTION deployment?", ok: "Deploy", submitter: "admin"
                        echo "Deploying to PRODUCTION..."
                        sh '''
                            mkdir -p deploy/prod
                            cp -r dist/* deploy/prod/
                            echo "PRODUCTION deployment completed."
                        '''
                    }
                    else {
                        echo "Branch '${branch}' is not configured for deployment."
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment pipeline for branch '${env.BRANCH_NAME}' completed successfully!"
        }
        failure {
            echo "❌ Deployment pipeline for branch '${env.BRANCH_NAME}' failed!"
        }
    }
}

