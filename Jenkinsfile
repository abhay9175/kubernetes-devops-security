pipeline {
  agent any
	
  environment {
    deploymentName = "devsecops"
    containerName = "devsecops-container"
    serviceName = "devsecops-svc"
    imageName = "abhaymarwade/devsecops_new:${GIT_COMMIT}"
    applicationURL="https://ec2-43.205.13.155.ap-south-1.compute.amazonaws.com"
    applicationURI="increment/99"
  }
	
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
        sh "mvn clean verify sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.projectName='numeric-application' -Dsonar.host.url=http://ec2-43-205-13-155.ap-south-1.compute.amazonaws.com:9000 -Dsonar.token=sqp_e5e1b7bdfbba326db926989c978375b172a04608"
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
                sh 'docker run --rm -v \$(pwd):/project openpolicyagent/conftest test --policy opa-docker-security.rego Dockerfile'
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

    stage('Vulnerability Scan - Kubernetes') {
      steps {
	parallel(
           "OPA Scan": {
             sh "docker run --rm -v \$(pwd):/project openpolicyagent/conftest test --policy opa-k8s-security.rego k8s_deployment_service.yaml"
           },
	   "Kubesec Scan": {
             sh "bash kubesec-scan.sh"
           },
	   "Trivy Scan": {
             sh "bash trivy-k8s-scan.sh"
          }
	)
      }
    }
	
    stage('K8S Deployment - DEV') {
      steps {
        parallel(
          "Deployment": {
            withKubeConfig([credentialsId: 'kubeconfig']) {
              sh "bash k8s-deployment.sh"
             }
          },
          "Rollout Status": {
            withKubeConfig([credentialsId: 'kubeconfig']) {
              sh "bash k8s-deployment-rollout-status.sh"
            }
          }
        )
      }
    }
	  
     stage('Integration Tests - DEV') {
       steps {
         script {
           try {
             withKubeConfig([credentialsId: 'kubeconfig']) {
               sh "bash integration-test.sh"
            }
          } catch (e) {
             withKubeConfig([credentialsId: 'kubeconfig']) {
               sh "kubectl -n default rollout undo deploy ${deploymentName}"
             }
             throw e
           }
         }
       }
     }
   }
	  
       #stage('OWASP ZAP - DAST') {
        #steps {
         #  withKubeConfig([credentialsId: 'kubeconfig']) {
          #   sh 'bash zap.sh'
           #}
         #}
       #}
    
	
    post { 
         always { 
           junit 'target/surefire-reports/TEST-com.devsecops.NumericApplicationTests.xml'
           jacoco execPattern: 'target/jacoco.exec'
           dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'	
	   publishHTML([allowMissing: false, alwaysLinkToLastBuild: true, keepAll: true, reportDir: 'owasp-zap-report', reportFiles: 'zap_report.html', reportName: 'OWASP ZAP HTML Report', reportTitles: 'OWASP ZAP HTML Report', useWrapperFileDirectly: true])
	}
      }
