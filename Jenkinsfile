// Define the URL of the Artifactory registry
def registry = 'https://trialvvnuv3.jfrog.io/'

pipeline {
    agent any

    environment {
        PATH = "/opt/maven/bin:$PATH"
        scannerHome = tool 'keshav-sonar-scanner'
    }

    stages {
        stage("build") {
            steps {
                echo "----------- build started ----------"
                sh 'mvn clean deploy -DskipTests'
                echo "----------- build completed ----------"
            }
        }

        stage("test") {
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
                    def server = Artifactory.newServer url: registry + "/artifactory", credentialsId: "artifact-cred"
                    def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}"
                    def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "jarstaging/(*)",
                              "target": "https://trialvvnuv3.jfrog.io/artifactory/sample-java-libs-release/",
                              "flat": "false",
                              "props": "${properties}",
                              "exclusions": [ "*.sha1", "*.md5"]
                            }
                         ]
                     }"""
                    def buildInfo = server.upload(uploadSpec)
                    buildInfo.env.collect()
                    server.publishBuildInfo(buildInfo)
                    echo '<--------------- Jar Publish Ended --------------->'
                }
            }
        }
    }
}
