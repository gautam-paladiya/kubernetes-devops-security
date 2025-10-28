pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true            }
        }   

      stage('Unit test - Jacoco') {
            steps {
              sh "mvn test"
            }

            post {
              always {
                junit 'target/surefire-reports/*.xml'
                jacoco executePattern: 'target/jacoco.exec'
              }
            }
        }   
    }
}