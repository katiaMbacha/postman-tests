pipeline {
  agent any

  triggers {
    githubPush()
    // Tous les jours à 12:00, heure de Paris
    cron('''TZ=Europe/Paris
0 12 * * *''')
  }

  tools {
    nodejs 'Node24' 
  }

  environment {
    REPORT_DIR = 'newman'
    EXO1 = 'collections/Exo1.postman_collection.json'
    EXO2 = 'collections/Exo2.postman_collection.json'
  }

  options {
    timestamps()
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
        sh 'rm -rf "$REPORT_DIR" && mkdir -p "$REPORT_DIR"'
      }
    }

    stage('Check Node & Newman') {
      steps {
        sh '''
          echo "📦 Versions installées"
          node -v
          npm -v
          npx newman -v
        '''
      }
    }

    stage('Run Exo1') {
      steps {
        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
          sh """
            echo "🚀 Exécution de Exo1"
            npx newman run "$EXO1" \
              -r cli,htmlextra,junit \
              --reporter-htmlextra-export "$REPORT_DIR/Exo1.html" \
              --reporter-junit-export     "$REPORT_DIR/Exo1.xml"
          """
        }
      }
    }

    stage('Run Exo2') {
      steps {
        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
          sh """
            echo "🚀 Exécution de Exo2"
            npx newman run "$EXO2" -d data/myClients.json \
              -r cli,htmlextra,junit \
              --reporter-htmlextra-export "$REPORT_DIR/Exo2.html" \
              --reporter-junit-export     "$REPORT_DIR/Exo2.xml"
          """
        }
      }
    }
  }

  post {
    always {
      echo '📊 Publication des rapports...'
      archiveArtifacts artifacts: 'newman/**', fingerprint: true

      publishHTML(target: [
        reportDir: 'newman',
        reportFiles: 'Exo1.html',
        reportName: '🟢 Rapport Exo1',
        keepAll: true,
        alwaysLinkToLastBuild: true
      ])
      publishHTML(target: [
        reportDir: 'newman',
        reportFiles: 'Exo2.html',
        reportName: '🔵 Rapport Exo2',
        keepAll: true,
        alwaysLinkToLastBuild: true
      ])

      junit testResults: 'newman/*.xml', allowEmptyResults: true

      script {
        emailext(
          to: 'katia.m.bacha@gmail.com',
          subject: "Jenkins · ${env.JOB_NAME} #${env.BUILD_NUMBER} · ${currentBuild.currentResult}",
          body: """
            <p>Résultat: <b>${currentBuild.currentResult}</b></p>
            <p>Job: ${env.JOB_NAME} (#${env.BUILD_NUMBER})</p>
            <p>Détails: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
          """,
          mimeType: 'text/html',
          attachLog: true,
          compressLog: true
        )
      }
    }
    success  { echo '✅ Tous les tests sont passés avec succès.' }
    unstable { echo '⚠️ Certains tests ont échoué — consulte les rapports.' }
    failure  { echo '❌ Échec critique du pipeline.' }
  }
}
