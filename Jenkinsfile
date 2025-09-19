pipeline {
    agent any


    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }


        stage('Backup') {
            steps {
                sh 'cp -r /var/www/html /var/www/html.backup || true'
            }
        }


        stage('Deploy') {
            steps {
                sh 'cp -r * /var/www/html/'
            }
        }


        stage('Test') {
            steps {
                sh 'curl -f http://localhost/ || true'
            }
        }
    }


    post {
        success {
            echo "Déploiement réussi du site web statique"
        }
        failure {
            echo "Erreur lors du déploiement"
        }
        always {
            sh 'rm -rf /var/www/html.backup || true'
            echo " Nettoyage effectué"
        }
    }
}
