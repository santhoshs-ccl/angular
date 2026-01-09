pipeline {
    agent any  // Use any available Jenkins node

    environment {
        PROJECT_NAME = "angular-app"
        NODE_VERSION = "18" // Change if your project needs another Node version
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '10')) // Keep last 10 builds
        timestamps() // Add timestamps to logs
    }

    stages {

        stage('Checkout SCM') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "*/${env.BRANCH_NAME}"]],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [[$class: 'CleanBeforeCheckout']],
                    userRemoteConfigs: [[
                        url: 'git@github.com:santhoshs-ccl/angular.git',
                        credentialsId: 'your-ssh-key-id' // Replace with your Jenkins SSH key ID
                    ]]
                ])
            }
        }

        stage('Install Node & NPM') {
            steps {
                sh '''
                    echo "Using Node.js version:"
                    node -v || nvm install $NODE_VERSION
                    npm install -g @angular/cli
                    npm ci
                '''
            }
        }

        stage('Run Lint') {
            steps {
                sh 'ng lint'
            }
        }

        stage('Build Angular') {
            steps {
                script {
                    // Build based on branch
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

        stage('Test Angular') {
            steps {
                sh 'npm test -- --watch=false --browsers=ChromeHeadless'
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'dist/**', allowEmptyArchive: true
            }
        }

        stage('Deploy') {
            when {
                anyOf {
                    branch 'main'
                    branch 'dev'
                    branch 'qa'
                }
            }
            steps {
                echo "Deploy steps for branch: ${env.BRANCH_NAME}"
                // Example: scp, rsync, or kubectl commands
                // sh "scp -r dist/ user@server:/var/www/${env.BRANCH_NAME}/"
            }
        }
    }

    post {
        success { echo "✅ Build & Deploy successful for ${env.BRANCH_NAME}" }
        failure { echo "❌ Build failed for ${env.BRANCH_NAME}" }
        always { cleanWs() } // Clean workspace after build
    }
}

