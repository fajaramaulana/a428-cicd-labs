node {
    env.CI = 'true'

    try {
        // Ensure Git repository is checked out
        checkout scm

        docker.image('node:16-buster-slim').inside {
            // Check the working directory inside the container
            sh 'pwd'  // Print the current directory
            sh 'ls -l'  // This will show all files, including package.json if it's there

            // Stage: Build
            stage('Build') {
                echo 'Installing dependencies...'
                sh 'npm install'
            }

            // Stage: Test
            stage('Test') {
                echo 'Running tests...'
                sh './jenkins/scripts/test.sh'
            }

            // Stage: Deploy
            stage('Deploy') {
                echo 'Running Deploy script...'
                sh './jenkins/scripts/deliver.sh'

                // Wait for user input
                input message: 'Sudah selesai menggunakan React App? (Klik "Proceed" untuk mengakhiri)'

                // Run kill script after input
                sh './jenkins/scripts/kill.sh'
            }
        }
    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    }
}
