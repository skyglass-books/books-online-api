pipeline {
  agent any
  def workspace = WORKSPACE

  stages {

    stage('Build Artifact - Maven') {
        agent { //here we select only docker build agents
            docker {
                image 'maven:latest' //container will start from this image
                args '-v /root/.m2:/root/.m2 -v ${workspace}/target:./target' //here you can map local maven repo, this let you to reuse local artifacts
            }
        }
        steps {
            sh 'mvn clean package -DskipTests=true' //this command will be executed inside maven container
            archive 'target/*.jar'
        }
    }

    stage('Unit Tests - JUnit and Jacoco') { //on this stage New container will be created, but current pipeline workspace will be remounted to it automatically
        agent {
            docker {
                image 'maven:latest'
                args '-v /root/.m2:/root/.m2'
            }
        }
        steps {
            sh 'mvn test' 
        }
        post {
          always {
            junit 'target/surefire-reports/*.xml'
            jacoco execPattern: 'target/jacoco.exec'
          }
        }        
    }   

    stage('Docker Build and Push') {
      steps {
        withDockerRegistry([credentialsId: "dockerHubCredentials", url: ""]) {
          sh 'printenv'
          sh 'docker build -t skyglass/numeric-app:""$GIT_COMMIT"" .'
          sh 'docker push skyglass/numeric-app:""$GIT_COMMIT""'
        }
      }
    }
  }
}