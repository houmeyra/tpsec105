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

    stage('Analyze with grype') {
      steps {
        script {
          try {
            sh 'set -o pipefail ; /usr/local/bin/grype -f high -q ${REPOSITORY}:${BUILD_NUMBER}'
          } catch (err) {
            // if scan fails, clean up (delete the image) and fail the build
            sh """
              echo "Vulnerabilities detected in ${REPOSITORY}:${BUILD_NUMBER}, cleaning up and failing build."
              docker rmi ${REPOSITORY}:${BUILD_NUMBER}
              exit 1
            """
          } // end try/catch
        } // end script
      } // end steps
    } // end stage "analyze with grype"
}
