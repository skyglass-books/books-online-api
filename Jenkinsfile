pipeline {
  agent any

  environment {
    deploymentName = "devsecops"
    containerName = "devsecops-container"
    serviceName = "devsecops-svc"
    imageName = "skyglass/numeric-app:${GIT_COMMIT}"
    applicationURL = "http://localhost:8080/"
    applicationURI = "/increment/99"
  }

  stages {

    stage('Build Artifact - Maven') {
      steps {
        sh "mvn clean package -DskipTests=true"
        archive 'target/*.jar'
      }
    }

    stage('Unit Tests - JUnit and Jacoco') {
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

    // stage('Mutation Tests - PIT') {
    //   steps {
    //     sh "mvn org.pitest:pitest-maven:mutationCoverage"
    //   }
    //   post {
    //     always {
    //       pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
    //     }
    //   }
    // }    

    stage('Docker Build and Push') {
      steps {
        withDockerRegistry([credentialsId: "dockerHubCredentials", url: ""]) {
          sh 'printenv'
          sh 'docker build -t skyglass/numeric-app:""$GIT_COMMIT"" .'
          sh 'docker push skyglass/numeric-app:""$GIT_COMMIT""'
        }
      }
    }

    // stage('K8S Deployment - DEV') {
    //   steps {
    //     parallel(
    //       "Deployment": {
    //         withKubeConfig([credentialsId: 'kubeconfig']) {
    //           sh "bash k8s-deployment.sh"
    //         }
    //       },
    //       "Rollout Status": {
    //         withKubeConfig([credentialsId: 'kubeconfig']) {
    //           sh "bash k8s-deployment-rollout-status.sh"
    //         }
    //       }
    //     )
    //   }
    // }

    stage('K8S Deployment - PROD') {
      steps {
        parallel(
          "Deployment": {
            withKubeConfig([credentialsId: 'kubeconfig']) {
              sh "sed -i 's#replace#${imageName}#g' k8s_PROD-deployment_service.yaml"
              sh "kubectl -n prod apply -f k8s_PROD-deployment_service.yaml"
            }
          },
          "Rollout Status": {
            withKubeConfig([credentialsId: 'kubeconfig']) {
              sh "bash k8s-PROD-deployment-rollout-status.sh"
            }
          }
        )
      }
    }

    // stage('Integration Tests - DEV') {
    //   steps {
    //     script {
    //       try {
    //         withKubeConfig([credentialsId: 'kubeconfig']) {
    //           sh "bash integration-test.sh"
    //         }
    //       } catch (e) {
    //         withKubeConfig([credentialsId: 'kubeconfig']) {
    //           sh "kubectl -n default rollout undo deploy ${deploymentName}"
    //         }
    //         throw e
    //       }
    //     }
    //   }
    // }

  }
}