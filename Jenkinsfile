pipeline {
  agent any
  options { timestamps() }

  environment {
    SCANCENTRAL_PATH = "${WORKSPACE}/bin/scancentral"
    JAVA_HOME        = "/usr/lib/jvm/java-17-openjdk-amd64"
    PATH             = "${JAVA_HOME}/bin:${env.PATH}"
  }

pipeline {
    agent any
    options { timestamps() }

    environment {
        SCANCENTRAL_PATH = "scancentral" // Must be in PATH
        JAVA_HOME        = "/usr/lib/jvm/java-17-openjdk-amd64"
        PATH             = "${JAVA_HOME}/bin:${env.PATH}"
    }

    stages {

        stage('Check .NET SDK') {
            steps {
                sh '''
                    echo "=== Checking .NET SDK Installation ==="
                    if command -v dotnet >/dev/null 2>&1; then
                        dotnet --version
                    else
                        echo "ERROR: .NET SDK not found."
                        exit 1
                    fi
                '''
            }
        }

        stage('Build .NET Project') {
            steps {
                sh '''
                    echo "=== Restoring NuGet packages ==="
                    dotnet restore Account_SkyPlus.sln

                    echo "=== Building in Release mode ==="
                    dotnet build Account_SkyPlus.sln -c Release --no-restore
                '''
            }
        }

        stage('Verify ScanCentral') {
            steps {
                sh '''
                    echo "=== Checking ScanCentral CLI Version ==="
                    ${SCANCENTRAL_PATH} -version
                '''
            }
        }

        stage('Package with ScanCentral (bt none)') {
            steps {
                sh '''
                    echo "=== Packaging source + binaries for FoD ==="
                    ${SCANCENTRAL_PATH} package \
                        -bt none \
                        -bf Account_SkyPlus.sln \
                        -o output.zip \
                        --include "**/*.cs" \
                        --include "**/*.vb" \
                        --include "**/bin/Release/**/*.dll" \
                        --include "**/bin/Release/**/*.exe" \
                        --include "**/bin/Release/**/*.pdb"
                    
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
 stage('Check MSBuild') {
  steps {
    sh '''
      echo "=== Checking MSBuild Installation ==="
      if command -v dotnet >/dev/null 2>&1; then
        echo "dotnet CLI found:"
        dotnet --version
        echo "MSBuild version:"
        dotnet msbuild -version
      else
        echo "ERROR: .NET SDK (with MSBuild) is not installed or not in PATH."
        echo "Please install .NET SDK before running this pipeline."
        exit 1
      fi
    '''
  }
}


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
        sh '''
  "$SCANCENTRAL_PATH" package \
    -bt none \
    -bf Account_SkyPlus.sln \
    -o output.zip
  ls -lh output.zip
'''

      }
    }

    stage('FoD Scan') {
      steps {
        script {
          fodStaticAssessment(
            releaseId: '1562867',         // Your FoD release ID
            releaseName: '1.0',           // Optional but kept
            scanCentral: 'none',       // Matches packaging tool
            srcLocation: "${WORKSPACE}/output.zip",
            overrideGlobalConfig: false   // Use FoD global credentials from Jenkins config
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
