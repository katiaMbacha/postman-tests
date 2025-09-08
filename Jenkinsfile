pipeline {
  agent any

  tools {
    nodejs 'Node24'   
  }

  parameters {
    choice(
      name: 'ENV',
      choices: ['dev', 'qa', 'test'],
      description: 'Choisissez l\'environnement Postman d'exécution'
    )
    string(
      name: 'DELAY_MS',
      defaultValue: '300',
      description: 'Délai entre requêtes (ms) pour éviter la saturation'
    )
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
          echo "📦 Vérification des versions installées"
          node -v
          npm -v
          newman -v
        '''
      }
    }

    stage('Run Exo1') {
      steps {
        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
          script {
            // On cherche un fichier d'environnement correspondant (ex: environments/dev.postman_environment.json)
            def envFile = "environments/${params.ENV}.postman_environment.json"
            def envArgs = fileExists(envFile) ? "-e ${envFile}" : ""

            // Fallback : si pas de fichier, on construit une baseUrl différente selon ENV
            if (!fileExists(envFile)) {
              def baseUrls = [
                dev : 'https://dev.example.com',
                qa  : 'https://qa.example.com',
                test: 'https://test.example.com'
              ]
              envArgs = "--env-var baseUrl=${baseUrls[params.ENV]}"
            }

            sh """
              echo "🚀 Exécution de Exo1 sur ENV=${params.ENV}"
              newman run "$EXO1" ${envArgs} \
                --delay-request ${params.DELAY_MS} \
                -r cli,htmlextra,junit \
                --reporter-htmlextra-export "$REPORT_DIR/Exo1.html" \
                --reporter-junit-export     "$REPORT_DIR/Exo1.xml"
            """
          }
        }
      }
    }

    stage('Run Exo2') {
      steps {
        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
          script {
            def envFile = "environments/${params.ENV}.postman_environment.json"
            def envArgs = fileExists(envFile) ? "-e ${envFile}" : ""

            if (!fileExists(envFile)) {
              def baseUrls = [
                dev : 'https://dev.example.com',
                qa  : 'https://qa.example.com',
                test: 'https://test.example.com'
              ]
              envArgs = "--env-var baseUrl=${baseUrls[params.ENV]}"
            }

            sh """
              echo "🚀 Exécution de Exo2 sur ENV=${params.ENV}"
              newman run "$EXO2" ${envArgs} \
                -d data/myClients.json \
                --delay-request ${params.DELAY_MS} \
                -r cli,htmlextra,junit \
                --reporter-htmlextra-export "$REPORT_DIR/Exo2.html" \
                --reporter-junit-export     "$REPORT_DIR/Exo2.xml"
            """
          }
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
    }
    success {
      echo '✅ Tous les tests sont passés avec succès.'
    }
    unstable {
      echo '⚠️ Certains tests ont échoué — consulte les rapports.'
    }
    failure {
      echo '❌ Échec critique du pipeline.'
    }
  }
}
