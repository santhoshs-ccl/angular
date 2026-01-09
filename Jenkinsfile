pipeline {
    agent {
        docker { 
            image 'node:18-alpine' 
            args '-u root:root' // run as root inside container if needed
        }
    }

    environment {
        // You can define environment variables here
        PROJECT_NAME = "angular-app"
    }

    options {
        skipDefaultCheckout(true) // We'll checkout manually
        timestamps()              // Adds timestamps in console log
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
                        credentialsId: 'your-ssh-key-id'
                    ]]
                ])
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm ci'  // clean install
            }
        }

        stage('Build Angular') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        echo "Building production..."
                        sh 'ng build --configuration=production'
                    } else if (env.BRANCH_NAME == 'dev') {
                        echo "Building development..."
                        sh 'ng build --configuration=development'
                    } else if (env.BRANCH_NAME == 'qa') {
                        echo "Building QA..."
                        sh 'ng build --configuration=qa'
                    } else {
                        echo "Unknown branch, using default build..."
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
                script {
                    if (env.BRANCH_NAME == 'main') {
                        echo "Deploying to Production..."
                        // Add your prod deploy commands here
                    } else if (env.BRANCH_NAME == 'dev') {
                        echo "Deploying to Development..."
                        // Add your dev deploy commands here
                    } else if (env.BRANCH_NAME == 'qa') {
                        echo "Deploying to QA..."
                        // Add your QA deploy commands here
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Build & Deploy successful for branch ${env.BRANCH_NAME}"
        }
        failure {
            echo "Build failed for branch ${env.BRANCH_NAME}"
        }
    }
}

