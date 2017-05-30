#!groovy
def majorVersionNumber = 1.0
def version  = "${majorVersionNumber}.${env.BUILD_NUMBER}"

node {
  stage('Checkout') {
     git url: 'https://github.com/fsc12/user-registration-V2.git'
     echo "Building version ${version} ...."
  }  
  stage('SonarQube analysis') {
    withSonarQubeEnv('sonar5.6') {
      // requires SonarQube Scanner for Maven 3.2+
       echo "Building with version ${version} ...."
      sh "mvn versions:set -DnewVersion=${version}"
      sh "mvn -f user-registration-application/pom.xml  org.sonarsource.scanner.maven:sonar-maven-plugin:3.2:sonar"
    }
  }  
  
  stage('Build') {
      sh "mvn versions:set -DnewVersion=${version}"
      sh "mvn -f user-registration-application/pom.xml clean package"
      step([$class: 'ArtifactArchiver', artifacts: '**/target/*.jar', fingerprint: true])
      step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
  }
  
  stage('Upload Repo') {
      def server = Artifactory.server "repo111"
      def uploadSpec = """{
          "files": [
              {
              "pattern": "user-registration-application/target/user*.jar",
              "target": "libs-release-local/${env.JOB_NAME}/",
              "recursive": "false"
              }
           ]
        }"""
      server.upload(uploadSpec)
  }
  
}
