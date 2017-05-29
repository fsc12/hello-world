#!groovy
node {
  stage('Checkout') {
     git url: 'https://github.com/fsc12/user-registration-V2.git'
    def v = version().${env.BUILD_NUMBER}
     if (v) {
       echo "Building version ${v} ...."
     }
  }  
  stage('SonarQube analysis') {
    withSonarQubeEnv('SonarQubeScanner3') {
      // requires SonarQube Scanner for Maven 3.2+
      sh 'mvn -f pom-commit.pom org.sonarsource.scanner.maven:sonar-maven-plugin:3.2:sonar'
    }
  }  
  
  stage('Build') {
      sh "mvn -f pom-commit.pom -B verify -Dmaven.test.failure.ignore verify"
      step([$class: 'ArtifactArchiver', artifacts: '**/target/*.jar', fingerprint: true])
      step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
  }
}
def version() {
  def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
  matcher ? matcher[0][1] : null
}
