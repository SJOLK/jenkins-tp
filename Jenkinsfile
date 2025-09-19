pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        echo "Checkout"
        checkout scm
      }
    }

    stage('Backup') {
      steps {
        echo "Backup"
        sh 'sudo mkdir -p /var/www/html'
        sh 'sudo cp -r /var/www/html /var/www/html.backup || true'
      }
    }

    stage('Deploy') {
      steps {
        // Si rsync est là, on l’utilise (propre et rapide), sinon fallback en cp
       echo "Deploy"
        sh '''
          if command -v rsync >/dev/null 2>&1; then
            sudo rsync -a --delete --exclude='.git' --exclude='Jenkinsfile' ./ /var/www/html/
          else
            sudo cp -r * /var/www/html/
            sudo rm -rf /var/www/html/.git || true
          fi
        '''
      }
    }

    stage('Test') {
      steps {
        // Échoue si le site ne répond pas (enlève -f si tu veux que ça n'échoue pas)
        echo "Test"
        sh 'curl -f http://localhost/'
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
      sh 'sudo rm -rf /var/www/html.backup || true'
      echo "Nettoyage effectué"
    }
  }
}
