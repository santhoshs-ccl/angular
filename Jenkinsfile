pipeline {
    agent any

    // Optional: keep last 10 builds
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
    }

    // Optional parameter for non-multibranch jobs
    parameters {
        string(name: 'BRANCH_TO_BUILD', defaultValue: 'dev', description: 'Branch to build & deploy')
    }

    stages {

        stage('Checkout') {
            steps {
                script {
                    // Detect branch from multibranch or parameter
                    def branch = env.BRANCH_NAME ?: params.BRANCH_TO_BUILD
                    echo "Checking out branch: ${branch}"

                    checkout([$class: 'GitSCM',
                        branches: [[name: "*/${branch}"]],
                        userRemoteConfigs: [[
                            url: 'git@github.com:santhoshs-ccl/angular.git',
                            credentialsId: 'your-ssh-key-id'  // Replace with your Jenkins SSH key credential
                        ]]
                    ])
                }
            }
        }

        stage('Install Node & Build') {
            steps {
                sh '''
                set -e

                # Load NVM
                export NVM_DIR="$HOME/.nvm"
                [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"

                # Check Node & NPM
                node -v
                npm -v

                # Install dependencies and build
                npm install
                npm run build
                '''
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def branch = env.BRANCH_NAME ?: params.BRANCH_TO_BUILD

                    if (branch == 'dev') {
                        echo "Deploying to DEV environment..."
                        sh '''
                        mkdir -p deploy/dev
                        cp -r dist/* deploy/dev/
                        echo "✅ DEV deployment completed."
                        '''
                    } else if (branch == 'qa') {
                        input message: "Approve QA deployment?", ok: "Deploy", submitter: "admin"
                        echo "Deploying to QA environment..."
                        sh '''
                        mkdir -p deploy/qa
                        cp -r dist/* deploy/qa/
                        echo "✅ QA deployment completed."
                        '''
                    } else if (branch == 'main') {
                        input message: "Final approval for PRODUCTION deployment?", ok: "Deploy", submitter: "admin"
                        echo "Deploying to PRODUCTION..."
                        sh '''
                        mkdir -p deploy/prod
                        cp -r dist/* deploy/prod/
                        echo "✅ PRODUCTION deployment completed."
                        '''
                    } else {
                        echo "Branch '${branch}' is not configured for deployment."
                    }
                }
            }
        }
    }

    post {
        success {
            echo "🎉 Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
        cleanup {
            cleanWs()
        }
    }
}

