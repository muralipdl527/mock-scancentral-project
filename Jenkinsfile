pipeline {
  agent any
  options { timestamps() }

  environment {
    SCANCENTRAL_PATH = "${WORKSPACE}/bin/scancentral"
    JAVA_HOME        = "/usr/lib/jvm/java-17-openjdk-amd64"
    PATH             = "${JAVA_HOME}/bin:${env.PATH}"
  }

  stages {
    stage('Check .NET SDK') {
      steps {
        sh '''
          echo "=== Checking .NET SDK ==="
          if command -v dotnet >/dev/null 2>&1; then
            dotnet --version
          else
            echo "ERROR: dotnet SDK not found in PATH"
            exit 1
          fi
        '''
      }
    }

    stage('Prep ScanCentral CLI') {
      steps {
        sh '''
          echo "=== Ensuring ScanCentral CLI is executable ==="
          if [ ! -f "${SCANCENTRAL_PATH}" ]; then
            echo "ERROR: ${SCANCENTRAL_PATH} not found. Ensure it exists in repo under bin/"
            exit 1
          fi
          chmod +x "${SCANCENTRAL_PATH}"
          "${SCANCENTRAL_PATH}" -version
        '''
      }
    }

    stage('Build .NET Project') {
      steps {
        sh '''
          echo "=== Restore & Build (Release) ==="
          dotnet restore Account_SkyPlus.sln
          dotnet build   Account_SkyPlus.sln -c Release --no-restore
        '''
      }
    }

    stage('Package for FoD (-bt none)') {
      steps {
        sh '''
            echo "=== Creating temp package dir ==="
            rm -rf fod_package
            mkdir -p fod_package/bin

            echo "=== Copying source files ==="
            find Account_SkyPlus -type f -name "*.cs" | xargs -I {} cp {} fod_package/

            echo "=== Detecting Release build output ==="
            FRAMEWORK_DIR=$(find Account_SkyPlus/bin/Release -maxdepth 1 -type d -name "net*" | head -n 1)
            if [ -z "$FRAMEWORK_DIR" ]; then
                echo "ERROR: No build output found in Release folder"
                exit 1
            fi
            echo "Found framework dir: $FRAMEWORK_DIR"

            echo "=== Copying compiled binaries ==="
            cp -r "$FRAMEWORK_DIR"/* fod_package/bin/

            echo "=== Packaging with ScanCentral ==="
            "${SCANCENTRAL_PATH}" package \
              -bt none \
              -bf fod_package/Account_SkyPlus.csproj \
              -o output.zip

            echo "=== Package contents ==="
            unzip -l output.zip | head -n 50
        '''
      }
    }

    stage('Upload to FoD') {
      steps {
        script {
          fodStaticAssessment(
            releaseId: '1562867',
            scanCentral: 'none',
            srcLocation: "${WORKSPACE}/output.zip",
            overrideGlobalConfig: false
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
