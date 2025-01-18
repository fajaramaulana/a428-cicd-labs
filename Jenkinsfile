node {
    // Menetapkan environment variable secara manual
    environment.CI = 'true'

    try {
        // Stage pertama: Build
        stage('Build') {
            echo 'Installing dependencies...'
            sh 'npm install'
        }

        // Stage kedua: Test
        stage('Test') {
            echo 'Running tests...'
            sh './jenkins/scripts/test.sh'
        }

        // Stage ketiga: Deliver
        stage('Deliver') {
            echo 'Running deliver script...'
            sh './jenkins/scripts/deliver.sh'

            // Menunggu input dari pengguna
            input message: 'Finished using the website? (Click "Proceed" to continue)'

            // Menjalankan kill script setelah input
            sh './jenkins/scripts/kill.sh'
        }

    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    }
}
