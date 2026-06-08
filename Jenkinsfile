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
            emailext(
                subject: "Rapport Build Jenkins - #${BUILD_NUMBER} : ${currentBuild.result}",
                body: '${BUILD_LOG, maxLines=450}',
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
