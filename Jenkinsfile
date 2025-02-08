node {
    env.CI = 'true'
    env.NODE_OPTIONS = '--max_old_space_size=1024'

    try {
        checkout scm

        stage('Prepare Environment') {
            echo "Using Node.js LTS image..."
            docker.image('node:lts-buster-slim').inside('-p 3000:3000') {
                sh 'pwd'
                sh 'ls -l'
            }
        }

        stage('Build') {
            echo 'Configuring NPM cache...'
            docker.image('node:lts-buster-slim').inside('-p 3000:3000') {
                echo 'Installing dependencies...'
                sh 'rm -rf node_modules'
                sh 'npm install --no-audit --no-optional --verbose'
            }
        }

        stage('Test') {
            echo 'Running tests...'
            docker.image('node:lts-buster-slim').inside('-p 3000:3000') {
                sh './jenkins/scripts/test.sh'
            }
        }

        stage('Manual Approval') {
            input message: 'Lanjutkan ke tahap Deploy?'
        }

        stage('Deploy') {
            echo 'Running deploy script...'
            docker.image('node:lts-buster-slim').inside('-p 3000:3000') {
                sh './jenkins/scripts/deliver.sh'
            }

            stage('Waiting 60s') {
                echo 'Waiting for 60 seconds...'
                sleep 60
            }

            // Install SSH client if not available
            stage('Install SSH Client') {
                echo "Checking and installing SSH client..."
                sh '''
                    if ! command -v scp &> /dev/null; then
                        sudo apt-get update && sudo apt-get install -y openssh-client
                    fi
                '''
            }

            // Securely transfer the build artifacts
            stage('Transfer Files to Remote Server') {
                withCredentials([sshUserPrivateKey(credentialsId: 'sencod-instance-ssh-key',
                                                   keyFileVariable: 'DEPLOY_KEY',
                                                   usernameVariable: 'DEPLOY_USER')]) {
                    echo "Transferring files..."
                    sh '''
                        scp -i "$DEPLOY_KEY" -o StrictHostKeyChecking=no -r dist/* ${DEPLOY_USER}@13.228.170.129:/var/www/html/
                    '''
                }
            }

            // Wait and execute cleanup
            stage('Cleanup') {
                sh './jenkins/scripts/kill.sh'
            }
        }
    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        echo "Build failed: ${e.message}"
        throw e
    }
}
