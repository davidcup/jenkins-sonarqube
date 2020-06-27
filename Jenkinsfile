node {
    def myGradleContainer = docker.image('gradle:jdk8-alpine')
    myGradleContainer.pull()

    stage('prep') {
        git url: 'https://github.com/wardviaene/gs-gradle.git'
    }

    stage('build') {
      myGradleContainer.inside("-v ${env.HOME}/.gradle:/home/gradle/.gradle") {
        sh 'cd complete && /opt/gradle/bin/gradle build'
      }
    }

    stage('sonar-scanner') {
      def sonarqubeScannerHome = tool name: 'SonarQubeScanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
      withCredentials([string(credentialsId: 'sonar-token', variable: 'sonarLogin')]) {
        sh "${sonarqubeScannerHome}/bin/sonar-scanner -e -Dsonar.host.url=http://10.0.0.11:9095 -Dsonar.login=${sonarLogin} -Dsonar.projectName=gs-gradle -Dsonar.projectVersion=${env.BUILD_NUMBER} -Dsonar.projectKey=GS -Dsonar.sources=complete/src/main/ -Dsonar.tests=complete/src/test/ -Dsonar.language=java -Dsonar.java.binaries=."
      }
    }
    
    stage("Quality Gate") {
       steps {
        script {
         timeout(time: 1, unit: 'MINUTES') {
          def qg = waitForQualityGate()
          if (qg.status == "ERROR") {
           echo "Failed Quality Gates";
           waitForQualityGate abortPipeline: true
          }
          if (qg.status == 'OK') {
           echo "Passed Quality Gates!";
          }
         }
        }
       }
    }
}
