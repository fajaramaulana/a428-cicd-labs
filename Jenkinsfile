node {
    env.CI = 'true'

    try {
        checkout scm

        stage('Prepare Environment') {
            // Use Docker volume to persist node_modules between builds
            docker.image('node:lts-buster-slim').inside('-p 3000:3000 -v $WORKSPACE/node_modules:/workspace/node_modules') {
                sh 'pwd'
                sh 'ls -l'
                
                stage('Build') {
                    echo 'Installing dependencies...'
                    sh 'npm cache clean --force'  // Clean npm cache
                    sh 'npm install --prefer-offline --no-audit --verbose'  // Faster install with detailed logs
                }

                stage('Test') {
                    echo 'Running tests...'
                    sh './jenkins/scripts/test.sh'
                }

                stage('Manual Approval') {
                    input message: 'Lanjutkan ke tahap Deploy?'
                }

                stage('Deploy') {
                    echo 'Running deploy script...'
                    sh './jenkins/scripts/deliver.sh'
                    sleep 60
                    sh './jenkins/scripts/kill.sh'
                }
            }
        }
    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    }
}
