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

    stage('Scan with grype') {
      steps {
        script {
          try {
            sh 'set -o pipefail ; /usr/local/bin/grype -f high -q houmeyra/tpsec105:latest'
          } catch (err) {
            // if scan fails, clean up (delete the image) and fail the build
            sh """
              echo "Vulnerabilities detected in houmeyra/tpsec105:latest, cleaning up and failing build."
              exit 1
            """
          } // end try/catch
        } // end script
      } // end steps
    } // end stage "analyze with grype"
}
