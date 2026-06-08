pipeline {
    agent any
    stages {

        stage('Positionnement sur le projet') {
            steps {
                sh 'pwd && ls'
            }
        }

        stage('Résolution des dépendances Maven') {
            steps {
                sh '''
                    mvn dependency:resolve -q || true
                    echo "Cache Maven peuplé."
                '''
            }
        }

        stage('Analyse SAST avec Trivy') {
            steps {
                sh '''
                    echo "========================================="
                    echo "      DEMARRAGE DU SCAN TRIVY"
                    echo "========================================="
                    trivy fs \
                      --scanners vuln,misconfig \
                      --skip-files src/main/resources/webgoat/static/js/libs/mode-java.js \
                      --format table \
                      . | tee rapport_trivy_webgoat.txt || true
                    echo "========================================="
                    echo "        FIN DU SCAN TRIVY"
                    echo "========================================="
                    ls -la rapport_trivy_webgoat.txt
                '''
            }
        }

    }

    post {
        always {
            archiveArtifacts artifacts: 'rapport_trivy_webgoat.txt',
                             allowEmptyArchive: true
            emailext(
                subject: "Rapport Trivy WebGoat - Build #${BUILD_NUMBER} : ${currentBuild.result}",
                body: """
                    Bonjour Awa,

                    Un commit sur WebGoat a automatiquement déclenché le pipeline Jenkins.

                    Statut     : ${currentBuild.result}
                    Build N°   : ${BUILD_NUMBER}
                    Projet     : WebGoat (OWASP)
                    Branche    : ${GIT_BRANCH}
                    Commit     : ${GIT_COMMIT}
                    Outil SAST : Trivy

                    Le rapport complet est disponible en pièce jointe.

                    Cordialement,
                    Jenkins
                """,
                attachmentsPattern: 'rapport_trivy_webgoat.txt',
                to: 'awaseck@esp.sn'
            )
        }
        success {
            echo 'Pipeline terminé avec succès.'
        }
        failure {
            echo 'Pipeline échoué. Vérifiez les logs.'
        }
    }
}
