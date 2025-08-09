pipeline {
  agent any
  options { timestamps() }

  environment {
    SCANCENTRAL_PATH = "${WORKSPACE}/bin/scancentral"
    JAVA_HOME        = "/usr/lib/jvm/java-17-openjdk-amd64"
    PATH             = "${JAVA_HOME}/bin:${env.PATH}"
  }

  stages {
    stage('Check MSBuild') {
      steps {
        sh '''
          echo "=== Checking MSBuild Installation ==="
          if command -v msbuild >/dev/null 2>&1; then
            echo "MSBuild found:"
            msbuild -version
          else
            echo "ERROR: MSBuild is not installed or not in PATH."
            echo "Please install MSBuild before running this pipeline."
            exit 1
          fi
        '''
      }
    }

    stage('Verify ScanCentral Version') {
      steps {
        sh """
          echo "=== Checking ScanCentral CLI Version ==="
          ${SCANCENTRAL_PATH} -version
        """
      }
    }

    stage('Package with ScanCentral') {
      steps {
        sh """
          echo "=== Packaging project with ScanCentral ==="
          ${SCANCENTRAL_PATH} package \
            -bt MSBuild \
            -bf Account_SkyPlus.sln \
            -o output.zip
          ls -lh output.zip
        """
      }
    }

    stage('FoD Scan') {
      steps {
        script {
          fodStaticAssessment(
            releaseId: '1562867',         // Your FoD release ID
            releaseName: '1.0',           // Optional but kept
            scanCentral: 'msbuild',       // Matches packaging tool
            srcLocation: "${WORKSPACE}",
            overrideGlobalConfig: false   // Use FoD global credentials from Jenkins config
          )
        }
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'output.zip', fingerprint: true
    }
  }
}
