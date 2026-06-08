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
            sh 'cat /var/lib/jenkins/jobs/webgoat-sast-auto/builds/${BUILD_NUMBER}/log | head -450 > build_report.txt || true'
            archiveArtifacts artifacts: 'build_report.txt',
                             allowEmptyArchive: true
            emailext(
                subject: "Rapport Build Jenkins - #${BUILD_NUMBER} : ${currentBuild.result}",
                body: """
                    Bonjour Awa,

                    Le pipeline Jenkins vient de se terminer.

                    Statut     : ${currentBuild.result}
                    Build N°   : ${BUILD_NUMBER}
                    Projet     : WebGoat (OWASP)
                    Branche    : ${GIT_BRANCH}
                    Commit     : ${GIT_COMMIT}
                    Outil SAST : Trivy

                    Les 450 premières lignes du rapport de build sont disponibles en pièce jointe.

                    Cordialement,
                    Jenkins
                """,
                attachmentsPattern: 'build_report.txt',
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
