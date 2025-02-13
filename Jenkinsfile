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

    stage('Scanning Image') {
        steps {
            script {
                sh """
                    # Installer Grype si ce n'est pas déjà fait
                    if ! command -v grype &> /dev/null
                    then
                        curl -sSfL https://raw.githubusercontent.com/anchore/grype/main/install.sh | sh
                    fi

                    # Scanner l'image Docker
                    grype houmeyra/tpsec105 --fail-on high
                """
            }
        }
    }
}
