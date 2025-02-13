node {
    def app

    stage('Clone repository') {
        // Clonage du dépôt Git
        checkout scm
    }

    stage('Build image') {
        // Construction de l'image Docker
        app = docker.build("houmeyra/tpsec105")
    }

    stage('Push image') {
        // Connexion au registre Docker Hub et push des tags
        docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
            app.push("${env.BUILD_NUMBER}")
            app.push("latest")
        }
    }

    stage('Scan image') {
        script {
            // Scanner l'image Docker avec Grype
            def scanOutput = sh(script: "grype houmeyra/tpsec105:latest", returnStdout: true).trim()

            // Affichage et sauvegarde du scan
            echo "Scan Output: ${scanOutput}"
            writeFile file: 'grype_scan_output.txt', text: scanOutput
        }
    }
}
