pipeline {
  agent any

  tools {
    nodejs 'Node24'   // 👈 correspond exactement au nom configuré dans Jenkins
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
        sh '''
          echo "🚀 Exécution de Exo1"
          newman run "$EXO1" -r cli,htmlextra,junit \
            --reporter-htmlextra-export "$REPORT_DIR/Exo1.html" \
            --reporter-junit-export     "$REPORT_DIR/Exo1.xml"
        '''
      }
    }

    stage('Run Exo2') {
      steps {
        sh '''
          echo "🚀 Exécution de Exo2"
          newman run "$EXO2" -r cli,htmlextra,junit \
            --reporter-htmlextra-export "$REPORT_DIR/Exo2.html" \
            --reporter-junit-export     "$REPORT_DIR/Exo2.xml"
        '''
      }
    }

    stage('Publish reports') {
      steps {
        archiveArtifacts artifacts: 'newman/**', fingerprint: true

        publishHTML(target: [
          reportDir: 'newman',
          reportFiles: 'Exo1.html,Exo2.html',
          reportName: 'Newman HTML Reports',
          keepAll: true,
          alwaysLinkToLastBuild: true
        ])

        junit testResults: 'newman/*.xml'
      }
    }
  }

  post {
    always {
      echo '✅ Build terminé.'
    }
    unsuccessful {
      echo '❌ Des tests ont échoué — consulte les rapports HTML/JUnit.'
    }
  }
}
