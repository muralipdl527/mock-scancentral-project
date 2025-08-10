pipeline {
  agent any
  options { timestamps() }

  parameters {
    file(name: 'PACKAGE_ZIP', description: 'Upload prebuilt FoD package (.zip) from your local machine')
    string(name: 'RELEASE_ID', defaultValue: '8422', description: 'FoD Release ID')
    string(name: 'TENANT_CODE', defaultValue: 'Goindigo_POV', description: 'FoD Tenant Code')
    string(name: 'USERNAME', defaultValue: 'ankit', description: 'FoD Username')
    password(name: 'PASSWORD', defaultValue: '', description: 'FoD Password')
    string(name: 'ENTITLEMENT_ID', defaultValue: '2', description: 'FoD Entitlement ID')
  }

  environment {
    // Adjust these to your FoD region (APAC example)
    FOD_PORTAL_URL = 'https://apac.fortify.com'
    FOD_API_URL    = 'https://api.apac.fortify.com'
    FOD_JAR_PATH   = '/app/FodUpload.jar' // Path where FodUpload.jar is stored on the agent
  }

  stages {
    stage('Receive Package') {
      steps {
        script {
          if (!params.PACKAGE_ZIP) {
            error "No PACKAGE_ZIP uploaded. Please attach your prebuilt package .zip."
          }
        }
        sh '''
          echo "=== Copying uploaded package to package.zip ==="
          rm -f package.zip
          cp "${PACKAGE_ZIP}" package.zip
          ls -lh package.zip
        '''
      }
    }

    stage('Validate Package Contents') {
      steps {
        sh '''
          echo "=== Listing contents of package.zip ==="
          unzip -l package.zip | head -n 30
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
