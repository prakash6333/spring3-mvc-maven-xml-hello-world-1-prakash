node {
    def NEXUS_VERSION = "nexus3"
    def NEXUS_PROTOCOL = "http"
    def NEXUS_URL = "54.215.73.110:8081"
    def NEXUS_REPOSITORY = "test"
    def NEXUS_CREDENTIALS_ID = "nexus"  // matches Jenkins credentials ID

    stage('Checkout Code') {
        echo "🔹 Cloning repository..."
        git branch: 'main', url: 'https://github.com/prakash6333/spring3-mvc-maven-xml-hello-world-1-prakash.git'
    }

    stage('Build with Maven') {
        echo "🔹 Building the project using Maven..."
        try {
            sh 'mvn -Dmaven.test.failure.ignore=true clean package'
        } finally {
            echo "📦 Archiving JUnit test results..."
            catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'
            }
        }
    }

    stage('Publish Artifact to Nexus') {
        echo "🚀 Preparing to upload artifacts to Nexus..."

        def pom = readMavenPom file: "pom.xml"
        def filesByGlob = findFiles(glob: "target/*.${pom.packaging}")

        if (filesByGlob.length == 0) {
            error "❌ No artifact found in target directory!"
        }

        def artifactPath = filesByGlob[0].path
        echo "✅ Found artifact: ${filesByGlob[0].name} at ${artifactPath}"

        nexusArtifactUploader(
            nexusVersion: NEXUS_VERSION,
            protocol: NEXUS_PROTOCOL,
            nexusUrl: NEXUS_URL,
            groupId: pom.groupId,
            version: "${BUILD_NUMBER}",   // version increments per build
            repository: NEXUS_REPOSITORY,
            credentialsId: NEXUS_CREDENTIALS_ID,   // 👈 Added authentication
            artifacts: [
                [
                    artifactId: pom.artifactId,
                    classifier: '',
                    file: artifactPath,
                    type: pom.packaging
                ],
                [
                    artifactId: pom.artifactId,
                    classifier: '',
                    file: "pom.xml",
                    type: "pom"
                ]
            ]
        )
    }

    stage('Post Build Actions') {
        echo "🗂 Archiving build artifacts..."
        archiveArtifacts artifacts: 'target/*.war', fingerprint: true
        echo "✅ Build and deployment successful! Artifact uploaded to Nexus at ${NEXUS_URL}/repository/${NEXUS_REPOSITORY}/"
    }
}
