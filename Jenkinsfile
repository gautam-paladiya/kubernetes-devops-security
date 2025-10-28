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
                jacoco execPattern: 'target/jacoco.exec'
              }
            }
        }   

      stage('Mutation Tests - PIT') {
            steps {
              sh "mvn org.pitest:pitest-maven:mutationCoverage"
            }

            post {
              always {
                pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
              }
            }
        }
      
      stage('Docker Build and Push') {
            steps {
              withDockerRegistry([credentialsId:"fb96da43-0b92-4ded-a601-d3b349e0dbbe", url:""]){
                sh "printenv"
                sh 'docker build -t gpaladiya/numeric-app:""$GIT_COMMIT"" .'
                sh 'docker push gpaladiya/numeric-app:""$GIT_COMMIT""'
              }
            }
        }   

      stage('Kubernetes deploy') {
            steps {
              withKubeConfig([credentialsId:"kubeconfig"]){
                sh "sed -i 's#replace#gpaladiya/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
                sh "kubectl apply -f k8s_deployment_service.yaml"
              }
            }
        }
    }
}