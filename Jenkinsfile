pipeline {

  environment {
    dockerimagename = "milstein/nodeapp:${env.BUILD_NUMBER}"
    dockerImage = ""
  }

  agent any

  stages {

    stage('Checkout Source') {
      steps {
        git branch: 'main', url: 'https://github.com/Milstein-Corp/nodeapp.git'
      }
    }

    stage('Build image') {
      steps{
        script {
          dockerImage = docker.build dockerimagename
        }
      }
    }

    stage('Pushing Image') {
      environment {
        registryCredential = 'dockerhublogin'
      }
      steps{
        script {
          docker.withRegistry('https://registry.hub.docker.com', registryCredential){
            dockerImage.push()
          }
        }
      }
    }

    stage('Docker Remove Image') {
      steps {
        sh "docker rmi -f ${dockerimagename}"
        sh "docker rmi -f registry.hub.docker.com/${dockerimagename}"
      }
    }

    stage('Deploying App to Kubernetes') {      
      steps {
        withKubeConfig([credentialsId: 'kubernetes']) {
          sh "sed -i 's/nodeapp:latest/nodeapp:${env.BUILD_NUMBER}/g' deploymentservice.yml"
          sh 'kubectl apply -f deploymentservice.yml'
        }
      }
    }
  }
}
