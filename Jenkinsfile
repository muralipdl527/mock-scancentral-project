pipeline {
  agent any
  options { timestamps() }

  environment {
    // FoD AMS region URLs
    FOD_PORTAL_URL = 'https://ams.fortify.com'
    FOD_API_URL    = 'https://api.ams.fortify.com'

    // Path to FoDUploader JAR
    FOD_JAR_PATH   = '/home/murali/Downloads/FodUpload.jar'

    // Path to your already-built ZIP on Jenkins agent
    LOCAL_PACKAGE  = '/home/murali/Downloads/dotnet_built_local.zip'

    // FoD settings that don't change often
    TENANT_CODE    = 'tam_team_test'
    RELEASE_ID     = '1562867'
    ENTITLEMENT_PREF = 'SingleScanOnly' // other options: SubscriptionOnly, SingleScanFirstThenSubscription, SubscriptionFirstThenSingleScan
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

          echo "=== Verifying presence of binaries (.dll/.exe/.pdb) ==="
          if ! unzip -l output.zip | grep -E '\\.(dll|exe|pdb)$' >/dev/null; then
            echo "ERROR: No dll/exe/pdb found in output.zip"
            exit 1
          fi

          echo "=== Verifying presence of .sln or .csproj ==="
          if ! unzip -l output.zip | grep -E '\\.(sln|csproj)$' >/dev/null; then
            echo "ERROR: No .sln or .csproj found in output.zip"
            exit 1
          fi
        '''
      }
    }

    stage('Upload to FoD (FoDUploader CLI)') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'fod-password', usernameVariable: 'FOD_USER', passwordVariable: 'FOD_PASSWORD')]) {
          sh """
            echo "=== Uploading to FoD ==="
            java -jar "${FOD_JAR_PATH}" \
              -z "${WORKSPACE}/output.zip" \
              -purl "${FOD_PORTAL_URL}" \
              -aurl "${FOD_API_URL}" \
              -tc "${TENANT_CODE}" \
              -uc "${FOD_USER}" "${FOD_PASSWORD}" \
              -rid "${RELEASE_ID}" \
              -ep "${ENTITLEMENT_PREF}"
          """
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
