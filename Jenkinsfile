pipeline {
  agent any
  options { timestamps() }

  environment {
    // FoD AMS region URLs
    FOD_PORTAL_URL = 'https://ams.fortify.com'
    FOD_API_URL    = 'https://api.ams.fortify.com'

    // Path to FoD uploader JAR
    FOD_JAR_PATH   = '/app/FodUpload.jar'  // Adjust if jar is elsewhere

    // Path to your prebuilt ZIP on the Jenkins agent
    LOCAL_PACKAGE  = '/home/murali/Downloads/dotnet_built_local.zip'

    // FoD parameters
    RELEASE_ID     = '8422'
    TENANT_CODE    = 'Goindigo_POV'
    USERNAME       = 'ankit'
    PASSWORD       = 'YOUR_PASSWORD'      // ⚠️ Replace or use Jenkins credentials
    ENTITLEMENT_ID = '2'
  }

  stages {
    stage('Copy Package') {
      steps {
        sh '''
          echo "=== Copying local ZIP to workspace ==="
          if [ ! -f "${LOCAL_PACKAGE}" ]; then
            echo "ERROR: File not found at ${LOCAL_PACKAGE}"
            exit 1
          fi
          cp "${LOCAL_PACKAGE}" package.zip
          ls -lh package.zip
        '''
      }
    }

    stage('Validate Package Contents') {
      steps {
        sh '''
          echo "=== Listing contents of package.zip ==="
          unzip -l package.zip | head -n 30

          echo "=== Checking for required binaries and sources ==="
          if ! unzip -l package.zip | grep -E '\\.(dll|exe|pdb)$' >/dev/null; then
            echo "ERROR: No dll/exe/pdb found in package.zip"
            exit 1
          fi
          if ! unzip -l package.zip | grep -E '\\.(sln|csproj)$' >/dev/null; then
            echo "ERROR: No .sln or .csproj found in package.zip"
            exit 1
          fi
        '''
      }
    }

    stage('Upload to FoD via FoDUploader CLI') {
      steps {
        sh """
          echo "=== Starting FoD Upload ==="
          java -jar "${FOD_JAR_PATH}" \
            -z "${WORKSPACE}/package.zip" \
            -purl "${FOD_PORTAL_URL}" \
            -aurl "${FOD_API_URL}" \
            -tc "${TENANT_CODE}" \
            -uc "${USERNAME}" \
            -up "${PASSWORD}" \
            -rid "${RELEASE_ID}" \
            -ep "${ENTITLEMENT_ID}"
        """
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'package.zip', fingerprint: true
    }
  }
}
