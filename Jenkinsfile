kpipeline {
    agent any

    environment {
        // NVM setup
        NVM_DIR = "${HOME}/.nvm"
        NODE_VERSION = 'v24.12.0'
        PATH = "${NVM_DIR}/versions/node/${NODE_VERSION}/bin:${env.PATH}"
    }

    options {
        skipDefaultCheckout() // we do manual checkout for clarity
        timestamps()
        ansiColor('xterm')
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Checking out source code..."
                checkout([$class: 'GitSCM',
                          branches: [[name: "*/${env.BRANCH_NAME}"]],
                          userRemoteConfigs: [[url: 'git@github.com:santhoshs-ccl/angular.git']]])
            }
        }

        stage('Setup Node') {
            steps {
                script {
                    echo "Setting up Node version ${NODE_VERSION} using NVM"
                    sh """
                        export NVM_DIR=${NVM_DIR}
                        [ -s "\$NVM_DIR/nvm.sh" ] && \. "\$NVM_DIR/nvm.sh"
                        nvm install ${NODE_VERSION}
                        nvm use ${NODE_VERSION}
                        node -v
                        npm -v
                    """
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm run test -- --watch=false --browsers=ChromeHeadless'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build -- --prod'
            }
        }

        stage('Deploy to DEV') {
            when {
                branch 'develop'
            }
            steps {
                echo "Deploying to DEV environment..."
                sh '''
                    mkdir -p deploy/dev
                    cp -r dist/* deploy/dev/
                '''
            }
        }

        stage('Deploy to PROD') {
            when {
                branch 'main'
                beforeInput true
            }
            steps {
                input message: "Approve PRODUCTION deployment?", 
                      ok: "Deploy",
                      submitter: "admin"

                echo "Deploying to PRODUCTION environment..."
                sh '''
                    mkdir -p deploy/prod
                    cp -r dist/* deploy/prod/
                '''
            }
        }

    }

    post {
        success {
            echo "Pipeline finished successfully!"
        }
        failure {
            echo "Pipeline failed. Please check the logs."
        }
    }
}

