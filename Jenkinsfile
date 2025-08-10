pipeline {
  agent any
  options { timestamps() }

  parameters {
    string(name: 'LOCAL_PACKAGE', defaultValue: '/home/murali/Downloads/dotnet_built_local.zip', description: 'Path to the prebuilt ZIP on the Jenkins agent')
    string(name: 'FOD_JAR_PATH', defaultValue: '/app/FodUpload.jar', description: 'Path to FodUpload.jar on the Jenkins agent')
    string(name: 'FOD_PORTAL_URL', defaultValue: 'https://ams.fortify.com', description: 'FoD portal URL')
    string(name: 'FOD_API_URL', defaultValue: 'https://api.ams.fortify.com', description: 'FoD API URL')
    string(name: 'TENANT_CODE', description: 'FoD tenant code')
    string(name: 'USERNAME', description: 'FoD username')
    password(name: 'PASSWORD', description: 'FoD password')
    string(name: 'RELEASE_ID', description: 'FoD release ID')
    string(name: 'ENTITLEMENT_ID', defaultValue: '2', description: 'FoD entitlement preference ID')
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
