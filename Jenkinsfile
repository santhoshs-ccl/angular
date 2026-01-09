pipeline {
    agent any

    environment {
        NVM_DIR = "${HOME}/.nvm"
        NODE_VERSION = "18.17.1"
    }

    options {
        skipDefaultCheckout(true)
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {

        stage('Checkout') {
            steps {
                script {
                    echo "Checking out branch: ${env.BRANCH_NAME}"
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: env.BRANCH_NAME]],
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

        stage('Setup Node') {
            steps {
                sh '''
                    export NVM_DIR="$HOME/.nvm"
                    [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
                    nvm install $NODE_VERSION --lts --no-progress
                    nvm use $NODE_VERSION
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
                sh 'npm run build -- --output-path=dist'
            }
        }

        stage('Approval for Production') {
            when {
                branch 'main'
            }
            steps {
                script {
                    echo "Waiting for approval to deploy MAIN branch to PROD..."
                    input message: "Approve deployment to Production?", ok: "Deploy"
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        echo "Deploying to PROD..."
                        sh './scripts/deploy-prod.sh'
                    } else if (env.BRANCH_NAME == 'develop') {
                        echo "Deploying to UAT..."
                        sh './scripts/deploy-uat.sh'
                    } else {
                        echo "Feature branch, deploying to DEV..."
                        sh './scripts/deploy-dev.sh'
                    }
                }
            }
        }

    }

    post {
        success {
            echo "Build and deployment for branch ${env.BRANCH_NAME} completed successfully."
        }
        failure {
            echo "Build failed for branch ${env.BRANCH_NAME}."
        }
        always {
            cleanWs()
        }
    }
}

