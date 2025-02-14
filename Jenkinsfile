node {
    def app
    def scanFile = "grype_scan_output-$(date +%d%m%Y).txt"

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
            // Exécuter le scan et capturer la sortie dans un fichier
            def scanStatus = sh(script: "grype houmeyra/tpsec105:latest > ${scanFile}", returnStatus: true)

            // Vérifier le statut du scan
            if (scanStatus != 0) {
                echo "❌ Scan Grype a détecté des vulnérabilités critiques !"
                error("Build échoué à cause de vulnérabilités détectées.")
            } else {
                echo "✅ Scan Grype terminé avec succès. Aucun problème critique détecté."
            }

            // Afficher la sortie du fichier
            sh "cat ${scanFile}"
        }
    }

    stage('Send Report') {
        script {
            // Vérifier si le fichier existe avant d'envoyer l'email
            if (fileExists(scanFile)) {
                echo "📩 Envoi du rapport de scan par email..."

                emailext subject: "Rapport de Scan Grype - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                         body: """Bonjour,\n
                                 Voici le rapport de scan Grype pour l'image Docker : houmeyra/tpsec105:latest\n
                                 Consultez le fichier attaché pour plus de détails.\n
                                 Cordialement,\n
                                 Votre pipeline Jenkins""",
                         recipientProviders: [[$class: 'DevelopersRecipientProvider']],
                         attachmentsPattern: scanFile
            } else {
                echo "⚠️ Le fichier ${scanFile} n'existe pas, email non envoyé."
            }
        }
    }
}
