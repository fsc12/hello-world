#!groovy
node {
  stage('Build') {
     git url: 'https://github.com/jglick/simple-maven-project-with-tests.git'
     def v = version()
     if (v) {
       echo "Building version ${v} ..."
     }
  }  
  stage('Test') {
      def mvnHome = tool 'M3'
      bat "${mvnHome}\\bin\\mvn -B verify -Dmaven.test.failure.ignore verify"
      step([$class: 'ArtifactArchiver', artifacts: '**/target/*.jar', fingerprint: true])
      step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
  }
}
def version() {
  def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
  matcher ? matcher[0][1] : null
}
