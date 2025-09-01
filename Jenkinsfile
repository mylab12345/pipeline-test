// Define the Artifactory base URL
def registry = 'https://trialvvnuv3.jfrog.io/artifactory'

pipeline {
    agent any

    environment {
        PATH = "/opt/maven/bin:$PATH"
        scannerHome = tool 'keshav-sonar-scanner'
    }

    stages {
        stage("Checkout") {
            steps {
                echo "----------- SCM Checkout Started ----------"
                checkout scm
                echo "----------- SCM Checkout Completed ----------"
            }
        }

        stage("Build") {
            steps {
                echo "----------- Build Started ----------"
                // Only build and package (no deploy from Maven)
                sh 'mvn clean package -DskipTests'
                echo "----------- Build Completed ----------"
            }
        }

        stage("Test") {
            steps {
                echo "----------- Unit Tests Started ----------"
                sh 'mvn test jacoco:report'
                echo "----------- Unit Tests Completed ----------"
            }
        }

        stage("SonarQube Analysis") {
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

                    // Upload spec (upload jar from target dir to repo)
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "target/*.jar",
                                "target": "sample-java-libs-release/",
                                "flat": true,
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
