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
        }   

      // stage('Mutation Tests - PIT') {
      //       steps {
      //         sh "mvn org.pitest:pitest-maven:mutationCoverage"
      //       }

      //       post {
      //         always {
      //           pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
      //         }
      //       }
      //   }
      
      stage('SonarQube') {
            steps {
              withSonarQubeEnv('SonarQube'){
                sh "mvn clean verify sonar:sonar \
                  -Dsonar.projectKey=numeric-application \
                  -Dsonar.projectName='numeric-application' \
                  -Dsonar.host.url=http://gpaladiya.westus2.cloudapp.azure.com:9000 \
                  -Dsonar.token=sqp_1eff2cf06920cfdc4c5c5c80897655425918b2f1"
              }
              timeout(time:2,unit:'MINUTES'){
                script{
                  waitForQualityGate abortPipeline:true
                }
              }
            }
        }

      stage('Vulnerability Scan') {
        steps {
          sh "mvn dependency-check:check"
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

      post {
        always {
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec'
          dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
        }

        // success {

        // }

        // failure {

        // }

      }
    }
}