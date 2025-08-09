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
