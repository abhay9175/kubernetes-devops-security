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
      post {
        always {
          junit 'target/surefire-reports/TEST-com.devsecops.NumericApplicationTests.xml'
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
    
    stage('sonarQube - SAST') {
      steps {
        "mvn clean verify sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.projectName='numeric-application' -Dsonar.host.url=http://ec2-65-2-75-221.ap-south-1.compute.amazonaws.com:9000 -Dsonar.token=sqp_8db9b8dea4a4d4c603301f9a54b852d3bad41288"
      }
    }

    stage('Docker Build and Push') {
      steps {
        withDockerRegistry([credentialsId: "dockerhub", url: "https://index.docker.io/v1/"]) {
          sh 'printenv'
          sh 'sudo docker build -t abhaymarwade/devsecops:""$GIT_COMMIT"" .'
          sh 'docker push abhaymarwade/devsecops:""$GIT_COMMIT""'
        }
      }
    }
    stage('K8S Deployment - PROD') {
	    steps {
        withKubeConfig([credentialsId: 'kubeconfig']) {
          sh "sed -i 's#replace#abhaymarwade/devsecops:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
          sh "kubectl apply -f k8s_deployment_service.yaml"
        }
      }
    }
  }
}   
