pipeline {
  agent any
  options { timestamps(); ansiColor('xterm') }

  environment {
    WEB_ROOT       = '/var/www/html'   // change if needed
    SITE_CHECK_URL = 'http://localhost/'
  }

  stages {
    stage('Checkout') {
      steps {
        echo 'Récupération du code depuis SCM'
        checkout scm
      }
    }

    stage('Deploy') {
      steps {
        script { env.BACKUP_DIR = "${WEB_ROOT}.backup-${env.BUILD_ID}" }
        sh '''#!/usr/bin/env bash
set -euo pipefail

echo "[INFO] Sauvegarde -> $BACKUP_DIR"
sudo mkdir -p "$WEB_ROOT"

# Backup current site if present
if [ -d "$WEB_ROOT" ]; then
  sudo rm -rf "$BACKUP_DIR" || true
  sudo cp -a "$WEB_ROOT" "$BACKUP_DIR"
fi

# Deploy repository files to web root (exclude SCM/Jenkinsfile)
if command -v rsync >/dev/null 2>&1; then
  sudo rsync -a --delete --exclude='.git' --exclude='Jenkinsfile' ./ "$WEB_ROOT"/
else
  sudo find . -maxdepth 1 -type f ! -name 'Jenkinsfile' -exec cp -f {} "$WEB_ROOT"/ \\;
  sudo cp -rf ./* "$WEB_ROOT"/ || true
  sudo rm -rf "$WEB_ROOT/.git" || true
fi

sudo chown -R www-data:www-data "$WEB_ROOT" || true
'''
      }
    }

    stage('Test') {
      steps {
        sh '''#!/usr/bin/env bash
set -euo pipefail

echo "[TEST] Vérification HTTP ${SITE_CHECK_URL}"
curl -fsS "${SITE_CHECK_URL}" >/dev/null
echo "[TEST] OK"
'''
      }
    }
  }

  post {
    success {
      echo 'Félicitations ! Déploiement terminé avec succès sur tous les environnements.'
      echo 'Les équipes ont été notifiées (placeholder).'
    }
    failure {
      echo 'Échec détecté, procédure de rollback automatique initiée.'
      sh '''#!/usr/bin/env bash
set -euo pipefail
if [ -n "${BACKUP_DIR:-}" ] && [ -d "${BACKUP_DIR}" ]; then
  echo "[ROLLBACK] Restauration depuis ${BACKUP_DIR}"
  if command -v rsync >/dev/null 2>&1; then
    sudo rsync -a --delete "${BACKUP_DIR}/" "${WEB_ROOT}/"
  else
    sudo cp -a "${BACKUP_DIR}/." "${WEB_ROOT}/"
  fi
else
  echo "[ROLLBACK] Aucune sauvegarde trouvée – restauration impossible."
fi
'''
      echo "L'équipe technique a été alertée (placeholder)."
    }
    always {
      sh '''#!/usr/bin/env bash
set -euo pipefail
if [ -n "${BACKUP_DIR:-}" ] && [ -d "${BACKUP_DIR}" ]; then
  sudo rm -rf "${BACKUP_DIR}"
fi
'''
      echo 'Nettoyage effectué.'
    }
  }
}
