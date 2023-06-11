

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

    stage('Mutation tests - PIT') {
       steps {
         sh "mvn org.pitest:pitest-maven:mutationCoverage"
       }
       post {
        always {
          pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
        }
       }
     }

    stage('SonarQube - SAST') {
      steps {
        withSonarQubeEnv('SonarQube'){
          sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.projectName='numeric-application'"
        }
        timeout(time: 2, unit: 'MINUTES'){
          script {
            waitForQualityGate abortPipeline: true
           }
          }
      }
    }

    stage('Vulnerability Scan - Docker ') {
      steps {
        sh "mvn dependency-check:check"
      }
      post {
        always {
          dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
        }
      }
    }

     stage('docker build and push') {
       steps {
	        withDockerRegistry([credentialsId: "docker_credentials", url: ""]) {
            sh 'printenv'
       	    sh 'docker build -t thiagodockerid/devsecops:""$GIT_COMMIT"" .'
	          sh 'docker push thiagodockerid/devsecops:""$GIT_COMMIT""' 
	      }		
      } 
    }
     stage('K8s deploymewnt - DEV') {
       steps {
	        withKubeConfig([credentialsId: 'kubeconfig']) {
            sh "sed -i 's#replace#thiagodockerid/devsecops:${GIT_COMMIT}#g' k8s_commands/k8s_deployment_service.yaml"
       	    sh "kubectl apply -f k8s_commands/k8s_deployment_service.yaml"
          }
        }
     }
  }
}
