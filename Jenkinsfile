#!groovy
def majorVersionNumber = 1.0
def version  = "${majorVersionNumber}.${env.BUILD_NUMBER}"
def buildInfo = Artifactory.newBuildInfo()

node {
  stage('Checkout') {
     git url: 'https://github.com/fsc12/user-registration-V2.git'
  }  
  stage('SonarQube analysis') {
    withSonarQubeEnv('sonar5.6') {
      sh "mvn versions:set -DnewVersion=${version}"
      sh "mvn -f user-registration-application/pom.xml  org.sonarsource.scanner.maven:sonar-maven-plugin:3.2:sonar"
    }
  }  
  
  
  stage("Quality Gate"){
// Check if coverate threshold is met, otherwise fail the job
            def response = httpRequest 'http://sonar111.westeurope.cloudapp.azure.com:9000/api/qualitygates/project_status?projectKey=com.ewolff:user-registration-application'
            def slurper = new groovy.json.JsonSlurper()
            def result = slurper.parseText(response.content)

            if (result.projectStatus.status == "ERROR") {
               currentBuild.result = 'FAILURE' 
               error "Pipeline aborted due to quality gate failure."
            }  
  }  
  
  stage('Build') {
      sh "mvn versions:set -DnewVersion=${version}"
      sh "mvn -f user-registration-application/pom.xml clean package"
      buildInfo.env.capture = true
      buildInfo.env.collect()
      junit allowEmptyResults: true, testResults: '**/target/**/TEST*.xml'
 
  }
  
  stage('Upload Repo') {
      def server = Artifactory.server "repo111"
      def uploadSpec = """{
          "files": [
              {
              "pattern": "user-registration-application/target/user*.jar",
              "target": "libs-release-local/${env.JOB_NAME}/${version}/",
              "recursive": "false"
              }
           ]
        }"""
      def buildInfoUpload = server.upload(uploadSpec)
      buildInfo.append(buildInfoUpload)
      server.publishBuildInfo(buildInfo)
      
      
  }
  
}

