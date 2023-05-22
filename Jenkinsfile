

pipeline {
  agent any
  stages {

     stage('Build Artifact - Maven') {
       steps {
         sh "mvn clean package -DskipTests=true"
         archive 'target/*.jar'
       }
     }
     
     stage('Unit test') {
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
     stage('docker build and push') {
       steps {
        sh 'printenv'
	sh 'docker build -t thiagodockerid/devsecops:""$GIT_COMMIT"" .'
	sh 'docker push thiagodockerid/devsecops:""$GIT_COMMIT""' 
	}		
      } 
   }
 }
}
