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
      publishHTML(target: [reportDir: 'newman', reportFiles: 'Exo1.html', reportName: '🟢 Rapport Exo1', keepAll: true, alwaysLinkToLastBuild: true])
      publishHTML(target: [reportDir: 'newman', reportFiles: 'Exo2.html', reportName: '🔵 Rapport Exo2', keepAll: true, alwaysLinkToLastBuild: true])
      junit testResults: 'newman/*.xml', allowEmptyResults: true
  
      script {
        def status = currentBuild.currentResult
        def buildUrl = env.BUILD_URL  // utilisera ton Jenkins URL (ngrok) si configurée dans Jenkins Location
        emailext(
          to: 'katia.m.bacha@gmail.com',
          from: 'katia.m.bacha@gmail.com',
          subject: "Jenkins · ${env.JOB_NAME} #${env.BUILD_NUMBER} · ${status}",
          mimeType: 'text/html',
          // 👉 Pièces jointes: les rapports HTML htmlextra
          attachmentsPattern: 'newman/Exo1.html,newman/Exo2.html',
          // 👉 Corps de mail avec liens directs vers les rapports publiés
          body: """
          <h2 style="margin:0 0 10px">Résultat : <span style="color:${status=='SUCCESS'?'#2e7d32':'#c62828'}">${status}</span></h2>
          <p>Job : <b>${env.JOB_NAME}</b> (#${env.BUILD_NUMBER})</p>
          <ul>
            <li>Console : <a href="${buildUrl}console">${buildUrl}console</a></li>
            <li>Tests JUnit : <a href="${buildUrl}testReport">${buildUrl}testReport</a></li>
            <li>Rapport Exo1 : <a href="${buildUrl}artifact/newman/Exo1.html">${buildUrl}artifact/newman/Exo1.html</a></li>
            <li>Rapport Exo2 : <a href="${buildUrl}artifact/newman/Exo2.html">${buildUrl}artifact/newman/Exo2.html</a></li>
          </ul>
          <p>Les fichiers <code>Exo1.html</code> et <code>Exo2.html</code> sont aussi joints à cet e-mail.</p>
          """,
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
