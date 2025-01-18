node {
    env.CI = 'true'

    try {
        docker.image('node:16-buster-slim').inside {
            // Print the current directory
            sh 'pwd'

            // Ensure package.json is present
            sh 'ls -l'

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

            // Stage: Deliver
            stage('Deliver') {
                echo 'Running deliver script...'
                sh './jenkins/scripts/deliver.sh'

                // Wait for user input
                input message: 'Finished using the website? (Click "Proceed" to continue)'

                // Run kill script after input
                sh './jenkins/scripts/kill.sh'
            }
        }
    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    }
}
