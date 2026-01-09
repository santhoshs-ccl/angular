pipeline {
    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
    }

    parameters {
        string(name: 'BRANCH_TO_BUILD', defaultValue: 'develop', description: 'Branch to build and deploy')
    }

    stages {

        stage('Checkout') {
            steps {
                script {
                    echo "Checking out branch: ${params.BRANCH_TO_BUILD}"
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: "*/${params.BRANCH_TO_BUILD}"]],
                        doGenerateSubmoduleConfigurations: false,
                        extensions: [[$class: 'CleanBeforeCheckout']],
                        userRemoteConfigs: [[
                            url: 'git@github.com:santhoshs-ccl/angular.git',
                            credentialsId: 'your-ssh-cred-id'
                        ]]
                    ])
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

        stage('Deploy to STAGING') {
            when {
                anyOf {
                    branch 'develop'
                    branch 'feature/*'
                }
            }
            steps {
                sh '''
                echo "Deploying to STAGING..."
                mkdir -p deploy/staging
                cp -r dist/* deploy/staging/
                echo "STAGING deployment completed."
                '''
            }
        }

        stage('Admin Approval for PRODUCTION') {
            when {
                branch 'main'
            }
            steps {
                input message: "Approve PRODUCTION deployment?",
                      ok: "Deploy",
                      submitter: "admin"
            }
        }

        stage('Deploy to PRODUCTION') {
            when {
                branch 'main'
            }
            steps {
                sh '''
                echo "Deploying to PRODUCTION..."
                mkdir -p deploy/prod
                cp -r dist/* deploy/prod/
                echo "PRODUCTION deployment completed."
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
    }
}

