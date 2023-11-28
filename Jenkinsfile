pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
            }
        }
       stage('Unit test ') {
            steps {
              sh "mvn test"
            }
        post{
            always {
              junit 'target/surefire-reports/TEST-com.devsecops.NumericApplicationTests.xml'
              jacoco execPattern: 'target/jacoco.exec'
            }
          }
        stage('Docker Build and Push') {
          steps {
          withDockerRegistry([credentialsId: "dockerhub", url: ""]) {
            sh 'printenv'
            sh 'sudo docker build -t abhaymarwde/DevSecOps-app:""$GIT_COMMIT"" .'
            sh 'docker push ahaymarwade/DevSecOps-app:""$GIT_COMMIT""'
            }
          }
        } 
    }
}
