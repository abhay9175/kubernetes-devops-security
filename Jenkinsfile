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

    stage('Docker Build and Push') {
      steps {
        withDockerRegistry([credentialsId: "dockerhub", url: "https://hub.docker.com/"]) {
          sh 'printenv'
          sh 'sudo docker build -t abhaymarwade/devsecops:""$GIT_COMMIT"" .'
          sh 'docker push abhaymarwade/devsecops:""$GIT_COMMIT""'
        }
      }
    }
  }
}
