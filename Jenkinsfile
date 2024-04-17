pipeline {
    agent any

    environment {
        SSH_CREDENTIALS_ID = 'site' // Replace with your actual SSH credentials ID
        SERVER_IP = 'ec2-54-152-195-130.compute-1.amazonaws.com'
        REMOTE_USER = 'ubuntu' // Change this to the appropriate non-root user
        GITHUB_REPO_URL = 'https://github.com/Mutuwa99/thato_pro.git'
    }

    stages {
        stage('Prepare') {
            steps {
                script {
                    // Clean workspace
                    deleteDir()

                    // Checkout the code from the GitHub repository
                    checkout([$class: 'GitSCM', branches: [[name: '*/main']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: GITHUB_REPO_URL]]])
                }
            }
        }

        stage('Test SSH Connection') {
            steps {
                script {
                    // Use SSH credentials to test the connection
                    withCredentials([file(credentialsId: SSH_CREDENTIALS_ID, variable: 'SSH_KEY')]) {
                        // Add the server's host key to known_hosts
                        sh "ssh-keyscan -H ${SERVER_IP} >> ~/.ssh/known_hosts"

                        // Test the connection
                        sh "ssh -i \$SSH_KEY -o StrictHostKeyChecking=no ${REMOTE_USER}@${SERVER_IP} 'echo Connection successful'"
                    }
                }
            }
        }

        stage('Deploy Static Website') {
            steps {
                script {
                    // Ensure remote directory exists
                   // sh "ssh -i \$SSH_KEY ${REMOTE_USER}@${SERVER_IP} 'mkdir -p /var/www/html/'"

                    // Deploy the code tohhhh the server using scp
                    withCredentials([file(credentialsId: SSH_CREDENTIALS_ID, variable: 'SSH_KEY')]) {
                        try {
                            sh "scp -i \$SSH_KEY -o StrictHostKeyChecking=no -r ./ ${REMOTE_USER}@${SERVER_IP}:/var/www/html/"
                            echo 'Deployment successful!'
                        } catch (Exception e) {
                            echo "Deployment failed: ${e.message}"
                            error 'Deployment failed!'
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline successful!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
