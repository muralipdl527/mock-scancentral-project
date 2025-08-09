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
            applicationName: 'test_jenkins1', 
            applicationType: '1', 
            assessmentType: '274', 
            attributes: '', 
            auditPreference: '2', 
            bsiToken: 'eyJ0ZW5hbnRJZCI6MzQ2LCJ0ZW5hbnRDb2RlIjoidGFtX3RlYW1fdGVzdCIsInJlbGVhc2VJZCI6MTU2Mjg2NywicGF5bG9hZFR5cGUiOiJBTkFMWVNJU19QQVlMT0FEIiwiYXNzZXNzbWVudFR5cGVJZCI6Mjc0LCJ0ZWNobm9sb2d5VHlwZSI6IkF1dG8gRGV0ZWN0IiwidGVjaG5vbG9neVR5cGVJZCI6MzIsInRlY2hub2xvZ3lWZXJzaW9uIjpudWxsLCJ0ZWNobm9sb2d5VmVyc2lvbklkIjpudWxsLCJhdWRpdFByZWZlcmVuY2UiOiJBdXRvbWF0ZWQiLCJhdWRpdFByZWZlcmVuY2VJZCI6MiwiaW5jbHVkZVRoaXJkUGFydHkiOmZhbHNlLCJpbmNsdWRlT3BlblNvdXJjZUFuYWx5c2lzIjpmYWxzZSwicG9ydGFsVXJpIjoiaHR0cHM6Ly9hbXMuZm9ydGlmeS5jb20iLCJhcGlVcmkiOiJodHRwczovL2FwaS5hbXMuZm9ydGlmeS5jb20iLCJzY2FuUHJlZmVyZW5jZSI6IlN0YW5kYXJkIiwic2NhblByZWZlcmVuY2VJZCI6MX0=', 
            businessCriticality: '3', 
            entitlementId: '13679', 
            entitlementPreference: '2', 
            frequencyId: '2', 
            inProgressBuildResultType: '', 
            inProgressScanActionType: '', 
            isMicroservice: false, 
            languageLevel: '', 
            microserviceName: '', 
            openSourceScan: '', 
            overrideGlobalConfig: false, 
            personalAccessToken: '', 
            releaseId: '1562867', 
            releaseName: '1.0', 
            remediationScanPreferenceType: '', 
            scanCentral: 'dotnet', 
            scanCentralBuildCommand: '', 
            scanCentralBuildFile: '', 
            scanCentralBuildToolVersion: '', 
            scanCentralExcludeFiles: '', 
            scanCentralIncludeTests: '', 
            scanCentralRequirementFile: '', 
            scanCentralSkipBuild: '', 
            scanCentralVirtualEnv: '', 
            sdlcStatus: '', 
            srcLocation: '"${WORKSPACE}"', 
            technologyStack: '1', 
            tenantId: 'tam_team_test', 
            username: 'muralipdl527'
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
