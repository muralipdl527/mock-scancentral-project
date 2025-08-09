pipeline {
  agent any
  options { timestamps() }

  environment {
    SCANCENTRAL_PATH = "${WORKSPACE}/bin/scancentral"
    JAVA_HOME        = "/usr/lib/jvm/java-17-openjdk-amd64"
    PATH             = "${JAVA_HOME}/bin:${env.PATH}"
  }

  stages {
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
            -bt none \
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
            applicationName: '', 
            applicationType: '', 
            assessmentType: '', 
            attributes: '', 
            auditPreference: '', 
            bsiToken: '', 
            businessCriticality: '', 
            entitlementId: '', 
            entitlementPreference: '', 
            frequencyId: '', 
            inProgressBuildResultType: '', 
            inProgressScanActionType: '', 
            isMicroservice: false, 
            languageLevel: '', 
            microserviceName: '', 
            openSourceScan: '', 
            overrideGlobalConfig: false, 
            personalAccessToken: '', 
            releaseId: '', 
            releaseName: '', 
            remediationScanPreferenceType: '', 
            scanCentral: '', 
            scanCentralBuildCommand: '', 
            scanCentralBuildFile: '', 
            scanCentralBuildToolVersion: '', 
            scanCentralExcludeFiles: '', 
            scanCentralIncludeTests: '', 
            scanCentralRequirementFile: '', 
            scanCentralSkipBuild: '', 
            scanCentralVirtualEnv: '', 
            sdlcStatus: '', 
            srcLocation: '', 
            technologyStack: '', 
            tenantId: '', 
            username: ''
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
