pipeline {
  agent any

  stages {
    stage('Build Artifact') {
      steps {
        sh "mvn clean package -DskipTests=true"
        archive 'target/*.jar' //so that they can be downloaded later
      }
    }

    stage('Unit test') {
      steps {
        sh "mvn test"
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
    
    stage('sonarQube - SAST') {
      steps { 
	withSonarQubeEnv('sonarqube') {     
        sh "mvn clean verify sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.projectName='numeric-application' -Dsonar.host.url=http://65.2.142.177:9000"
      }
      timeout(time: 2, unit: 'MINUTES') {
        script {
          waitForQualityGate abortPipeline: true
        }
      }       
    }
  }

    stage('Vulnerability Scan - Docker') {
      steps {
	 parallel(
             "Dependency Scan": {
       	         sh "mvn dependency-check:check"
		      },
	     "Trivy Scan":{
		 sh "bash trivy-docker-image-scan.sh"
                      },
	     "OPA Conftest":{
		 sh "docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-docker-security.rego Dockerfile"
	 }   
        )
      }
    }
		
    stage('Docker Build and Push') {
      steps {
        withDockerRegistry([credentialsId: "dockerhub", url: "https://index.docker.io/v1/"]) {
          sh 'printenv'
          sh 'sudo docker build -t abhaymarwade/devsecops_new:""$GIT_COMMIT"" .'
          sh 'docker push abhaymarwade/devsecops_new:""$GIT_COMMIT""'
        }
      }
    }
    stage('K8S Deployment - PROD') {
	    steps {
        withKubeConfig([credentialsId: 'kubeconfig']) {
          sh "sed -i 's#replace#abhaymarwade/devsecops_new:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
          sh "kubectl apply -f k8s_deployment_service.yaml"
          }
        }
      }
    }
	  
    post { 
         always { 
           junit 'target/surefire-reports/TEST-com.devsecops.NumericApplicationTests.xml'
           jacoco execPattern: 'target/jacoco.exec'
           dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'		
	}
      }
    }   
