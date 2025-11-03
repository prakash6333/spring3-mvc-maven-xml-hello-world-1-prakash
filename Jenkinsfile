node {

    def mvnHome
    def NEXUS_VERSION = "nexus3"
    def NEXUS_PROTOCOL = "http"
    def NEXUS_URL = "54.215.73.110:8081"         // ✅ Your actual Nexus server
    def NEXUS_REPOSITORY = "test"                // ✅ Your hosted repository name
    def NEXUS_CREDENTIAL_ID = "nexus"            // ✅ Matches Jenkins credential ID

    stage('Checkout Code') {
        echo "Cloning repository..."
        git branch: 'main', url: 'https://github.com/prakash6333/spring3-mvc-maven-xml-hello-world-1-prakash.git'
        mvnHome = tool 'Maven'
    }

    stage('Build with Maven') {
        echo "Building the project using Maven..."
        withEnv(["MVN_HOME=$mvnHome"]) {
            if (isUnix()) {
                sh 'mvn -Dmaven.test.failure.ignore=true clean package'
            } else {
                bat(/"%MVN_HOME%\\bin\\mvn" -Dmaven.test.failure.ignore=true clean package/)
            }
        }
    }

    stage('Publish Artifact to Nexus') {
        echo "Preparing to upload artifacts to Nexus..."

        script {
            // Read Maven POM details
            def pom = readMavenPom file: "pom.xml"
            def filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
            
            if (filesByGlob.length == 0) {
                error "*** No artifacts found in target/ directory"
            }

            def artifactPath = filesByGlob[0].path
            def artifactExists = fileExists artifactPath

            if (artifactExists) {
                echo "*** Uploading ${artifactPath} to Nexus"
                echo "*** Group: ${pom.groupId}, Artifact: ${pom.artifactId}, Version: ${BUILD_NUMBER}"

                nexusArtifactUploader(
                    nexusVersion: NEXUS_VERSION,
                    protocol: NEXUS_PROTOCOL,
                    nexusUrl: NEXUS_URL,
                    groupId: pom.groupId,
                    version: "${BUILD_NUMBER}",        // ✅ Use Jenkins build number dynamically
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
            } else {
                error "*** File: ${artifactPath}, could not be found"
            }
        }
    }
}
