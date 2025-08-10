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
          echo "=== Restoring NuGet packages ==="
          dotnet restore Account_SkyPlus.sln

          echo "=== Building solution in Release mode ==="
          dotnet build Account_SkyPlus.sln -c Release --no-restore
        '''
      }
    }

    stage('Package for FoD (-bt none)') {
      steps {
        sh '''
            echo "=== Creating temporary package directory ==="
            rm -rf fod_package
            mkdir -p fod_package/bin

            echo "=== Copying all C# source files ==="
            find . -type f -name "*.cs" | grep -v scancentral | xargs -I {} cp --parents {} fod_package/

            echo "=== Searching and copying compiled binaries (dll/exe/pdb) ==="
            BIN_FILES=$(find . -type f -name "*.dll" -o -name "*.exe" -o -name "*.pdb" | grep -v scancentral || true)
            if [ -n "$BIN_FILES" ]; then
                echo "$BIN_FILES" | xargs -I {} cp --parents {} fod_package/
            else
                echo "ERROR: No real binaries found! Ensure DummyProject is added to the solution."
                exit 1
            fi

            echo "=== Packaging with ScanCentral CLI ==="
            "${SCANCENTRAL_PATH}" package \
              -bt none \
              -bf fod_package/Account_SkyPlus.csproj \
              -o output.zip

            echo "=== Verifying packaged binaries in output.zip ==="
            unzip -l output.zip | grep -E "\\.dll|\\.exe|\\.pdb" || true
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
