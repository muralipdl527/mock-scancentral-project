pipeline {
  agent any
  options { timestamps() }

  environment {
    // FoD AMS region URLs
    FOD_PORTAL_URL = 'https://ams.fortify.com'
    FOD_API_URL    = 'https://api.ams.fortify.com'

    // Path to FoDUploader JAR (updated to your location)
    FOD_JAR_PATH   = '/home/murali/Downloads/FodUpload.jar'

    // Path to your already-built ZIP on Jenkins agent
    LOCAL_PACKAGE  = '/home/murali/Downloads/dotnet_built_local.zip'
  }

  stages {
    stage('Copy Local Package') {
      steps {
        sh '''
          echo "=== Copying prebuilt ZIP to workspace ==="
          if [ ! -f "${LOCAL_PACKAGE}" ]; then
            echo "ERROR: File not found at ${LOCAL_PACKAGE}"
            exit 1
          fi
          cp "${LOCAL_PACKAGE}" output.zip
          ls -lh output.zip
        '''
      }
    }

    stage('Validate Package') {
      steps {
        sh '''
          echo "=== Checking ZIP contents ==="
          unzip -l output.zip | head -n 30

          echo "=== Verifying presence of binaries ==="
          if ! unzip -l output.zip | grep -E '\\.(dll|exe|pdb)$' >/dev/null; then
            echo "ERROR: No dll/exe/pdb found in output.zip"
            exit 1
          fi
        '''
      }
    }

    stage('Upload to FoD (FoDUploader CLI)') {
      steps {
        sh """
          echo "=== Uploading to FoD ==="
          java -jar "${FOD_JAR_PATH}" \
            -z "${WORKSPACE}/output.zip" \
            -purl "${FOD_PORTAL_URL}" \
            -aurl "${FOD_API_URL}" \
            -tc "tam_team_test" \
            -uc "muralipdl57" \
            -up "YOUR_PASSWORD" \
            -rid "1562867" \
            -ep "2"
        """
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'output.zip', fingerprint: true
    }
  }
}
