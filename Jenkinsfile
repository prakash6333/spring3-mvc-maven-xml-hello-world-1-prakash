node {

    def mvnHome
    def NEXUS_VERSION = "nexus3"
    def NEXUS_PROTOCOL = "http"
    def NEXUS_URL = "54.215.73.110:8081"         // Nexus server URL
    def NEXUS_REPOSITORY = "test"                // Nexus repository name
    def NEXUS_CREDENTIAL_ID = "nexus"            // Jenkins credential ID

    stage('Checkout Code') {
        echo "🔹 Cloning repository..."
        git branch: 'main', url: 'https://github.com/prakash6333/spring3-mvc-maven-xml-hello-world-1-prakash.git'
    }

    stage('Setup Maven') {
        echo "🔹 Checking Maven installation..."

        // Try to use Maven tool from Jenkins if available
        try {
            mvnHome = tool 'Maven'
            echo "✅ Found Maven tool configured in Jenkins."
        } catch (Exception e) {
            echo "⚠️ Maven tool not found in Jenkins, installing manually..."
            sh '''
                if ! command -v mvn &> /dev/null; then
                    echo "Installing Maven..."
                    sudo apt-get update -y
                    sudo apt-get install -y maven
                else
                    echo "Maven already installed in the system."
                fi
            '''
            mvnHome = '/usr/share/maven'
        }

        sh 'mvn -version'
    }

    stage('Build with Maven') {
        echo "🔹 Building the project..."
        sh 'mvn -Dmaven.test.failure.ignore=true clean package'
    }

    stage('Publish Artifact to Nexus') {
        echo "🔹 Preparing to upload artifacts to Nexus..."

        script {
            // Read Maven POM details
            def pom = readMavenPom file: "pom.xml"
            def filesByGlob = findFiles(glob: "target/*.${pom.packaging}")

            if (filesByGlob.length == 0) {
                error "*** No artifacts found in target/ directory"
            }

            def artifactPath = filesByGlob[0].path
            echo "*** Uploading ${artifactPath} to Nexus"
            echo "*** Group: ${pom.groupId}, Artifact: ${pom.artifactId}, Version: ${BUILD_NUMBER}"

            nexusArtifactUploader(
                nexusVersion: NEXUS_VERSION,
                protocol: NEXUS_PROTOCOL,
                nexusUrl: NEXUS_URL,
                groupId: pom.groupId,
                version: "${BUILD_NUMBER}",
                repository: NEXUS_REPOSITORY,
                credentialsId: NEXUS_CREDENTIAL_ID,
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
    }
}
