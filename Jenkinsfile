node {
    env.CI = 'true'

    try {
        checkout scm

        stage('Prepare Environment') {
            docker.image('node:lts-buster-slim').inside('-p 3000:3000 -it') {
                sh 'pwd'
                sh 'ls -l'
                stage('Build') {
                    echo 'Installing dependencies...'
                    sh 'rm -rf node_modules package-lock.json && npm install'
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
