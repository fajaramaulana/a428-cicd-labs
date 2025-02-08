node {
    env.CI = 'true'
    env.NODE_OPTIONS = '--max_old_space_size=1024'

    try {
        checkout scm

        docker.image('node:lts-buster-slim').inside('-u root -p 3000:3000') {
            stage('Prepare Environment') {
                echo 'Using Node.js LTS image...'
                sh 'pwd'
                sh 'ls -l'
            }

            stage('Build') {
                echo 'Configuring NPM cache...'
                echo 'Installing dependencies...'
                sh 'rm -rf node_modules'
                sh 'npm install --no-audit --no-optional --verbose'
            }

            stage('Test') {
                echo 'Running tests...'
                sh './jenkins/scripts/test.sh'
            }

            stage('Deliver') {
                echo 'Running deploy script...'
                sh './jenkins/scripts/deliver.sh'
                echo 'Waiting for 60 seconds'
                sleep 60
            }

            stage('Manual Approval') {
                input message: 'Lanjutkan ke tahap Deploy?'
            }

            stage('Deploy') {
                stage('Ensure SSH Client is Installed') {
                    echo 'Checking and installing SSH client...'
                    sh '''
                        if ! command -v scp &> /dev/null; then
                            apt-get update && apt-get install -y openssh-client
                        fi
                    '''
                }

                stage('Transfer Files to Remote Server') {
                    withCredentials([
                        sshUserPrivateKey(credentialsId: 'second-instance-ssh-key',
                                          keyFileVariable: 'DEPLOY_KEY',
                                          usernameVariable: 'DEPLOY_USER'),
                        string(credentialsId: 'SECOND_INSTANCE_IP', variable: 'DEPLOY_HOST')
                    ]) {
                        echo "Transferring files to \$DEPLOY_HOST..."
                        sh '''
                            scp -i "$DEPLOY_KEY" -o StrictHostKeyChecking=no -r build/* ${DEPLOY_USER}@$DEPLOY_HOST:/home/${DEPLOY_USER}/var/www/html/
                        '''
                    }
                }

                stage('Cleanup') {
                    echo 'Cleaning up...'
                    sh './jenkins/scripts/kill.sh'
                }
            }
        }
    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        echo "Build failed: ${e.message}"
        throw e
    }
}
