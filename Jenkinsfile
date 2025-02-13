node {
    def app

    stage('Clone repository') {
        /* Let's make sure we have the repository cloned to our workspace */

        checkout scm
    }

    stage('Build image') {
        /* This builds the actual image; synonymous to
         * docker build on the command line */

        app = docker.build("houmeyra/tpsec105")
    }

    stage('Push image') {
        /* Finally, we'll push the image with two tags:
         * First, the incremental build number from Jenkins
         * Second, the 'latest' tag.
         * Pushing multiple tags is cheap, as all the layers are reused. */
        docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
            app.push("${env.BUILD_NUMBER}")
            app.push("latest")
        }
    }

    stage('Scan image') {
        steps {
            script {
                // Scanner l'image Docker depuis Docker Hub avec Grype
                def scanOutput = sh(script: "grype houmeyra/tpsec105:latest", returnStdout: true).trim()
 
                    // Afficher la sortie du scan
                    echo "Scan Output: ${scanOutput}"
 
                    // Sauvegarder la sortie dans un fichier
                    writeFile file: 'grype_scan_output.txt', text: scanOutput
          }
       }
    }
}
