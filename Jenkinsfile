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
        sh "mvn clean verify sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.projectName='numeric-application' -Dsonar.host.url=http://ec2-65.2.142.177.ap-south-1.compute.amazonaws.com:9000 -Dsonar.token=sqp_8db9b8dea4a4d4c603301f9a54b852d3bad41288"
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
  }
}   
