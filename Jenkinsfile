pipeline {
  agent any

  options {
    timestamps()
    ansiColor('xterm')
  }

  environment {
    REPORT_DIR = 'newman'
    EXO1 = 'collections/Exo1.postman_collection.json'
    EXO2 = 'collections/Exo2.postman_collection.json'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
        sh 'rm -rf "$REPORT_DIR" && mkdir -p "$REPORT_DIR"'
      }
    }

    stage('Check Node & npm') {
      steps {
        sh '''
          set -e
          node -v
          npm -v
        '''
      }
    }

    stage('Install Newman & reporter') {
      steps {
        sh '''
          set -e
          npm i -g newman newman-reporter-htmlextra
          newman -v
        '''
      }
    }

    stage('Run Exo1') {
      steps {
        sh '''
          set -e
          newman run "$EXO1" -r cli,htmlextra,junit \
            --reporter-htmlextra-export "$REPORT_DIR/Exo1.html" \
            --reporter-junit-export     "$REPORT_DIR/Exo1.xml"
        '''
      }
    }

    stage('Run Exo2') {
      steps {
        sh '''
          set -e
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
      echo 'Build terminé. Rapports archivés et publiés.'
    }
    unsuccessful {
      echo 'Des tests ont échoué — consulte les rapports HTML/JUnit.'
    }
  }
}
