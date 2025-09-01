// Define the URL of the Artifactory registry
def registry = 'https://trialvvnuv3.jfrog.io/artifactory'

pipeline {
    agent any

    environment {
        PATH = "/opt/maven/bin:$PATH"
        scannerHome = tool 'keshav-sonar-scanner'
    }

    stages {
        stage("Build") {
            steps {
                echo "----------- build started ----------"
                sh 'mvn clean deploy -DskipTests'
                echo "----------- build completed ----------"
            }
        }

        stage("Test") {
            steps {
                echo "----------- unit test started ----------"
                sh 'mvn test jacoco:report'
                echo "----------- unit test completed ----------"
            }
        }

        stage("SonarQube analysis") {
            steps {
                withSonarQubeEnv('keshav-sonarqube-server') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                              -Dsonar.login=$SONAR_TOKEN
                        """
                    }
                }
            }
        }

        stage("Jar Publish") {
            steps {
                script {
                    echo '<--------------- Jar Publish Started --------------->'

                    // Define Artifactory server
                    def server = Artifactory.newServer(
                        url: registry,
                        credentialsId: "artifact-cred"
                    )

                    // Build properties
                    def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}"

                    // Upload spec (use repo name, not full URL)
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "jarstaging/(*)",
                                "target": "sample-java-libs-release/",
                                "flat": false,
                                "props": "${properties}",
                                "exclusions": [ "*.sha1", "*.md5" ]
                            }
                        ]
                    }"""

                    def buildInfo = server.upload(uploadSpec)
                    buildInfo.env.capture = true
                    server.publishBuildInfo(buildInfo)

                    echo '<--------------- Jar Publish Ended --------------->'
                }
            }
        }
    }
}
