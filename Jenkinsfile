node {
    env.CI = 'true'

    try {
        // Ensure Git repository is checked out
        checkout scm

        stage('Prepare Environment') {
            docker.image('node:lts-buster-slim').inside('-p 3000:3000 -it') {
                sh 'pwd'  // Print the current directory
                sh 'ls -l'  // This will show all files, including package.json if it's there
                stage('Build') {
                    echo 'Installing dependencies...'
                    sh 'rm -rf node_modules package-lock.json && npm install'
                }

                stage('Test') {
                    echo 'Running tests...'
                    sh './jenkins/scripts/test.sh'
                }

                stage('Deliver') {
                    echo 'Running deploy script...'
                    sh './jenkins/scripts/deliver.sh'

                    input message: 'Sudah selesai menggunakan React App? (Klik "Proceed" untuk mengakhiri)'

                    sh './jenkins/scripts/kill.sh'
                }
            }
        }
    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    }
}
