pipeline {
  agent any
  environment {
    SCANCENTRAL_PATH = "${WORKSPACE}/bin/scancentral"
    JAVA_HOME = "/usr/lib/jvm/java-17-openjdk-amd64"
    PATH = "${JAVA_HOME}/bin:${env.PATH}"
  }
  stages {
    stage('Verify ScanCentral Version') {
      steps {
        sh '''
          echo "=== Checking version ==="
          ${SCANCENTRAL_PATH} -version
        '''
      }
    }
    stage('Run Scan (expected to fail for reproduction)') {
      steps {
        sh '''
    echo === Running scan step (will fail intentionally) ===
    ./bin/scancentral package -bt none -bf Account_SkyPlus.sln -o output.zip
'''
      }
    }
  }
}
