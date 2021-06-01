pipeline{
  agent any
  tools{
        maven 'maven3'
    }

  stages{
    stage('Maven Build'){
      steps{
        echo "${getLatestCommitId()}"
        sh "mvn clean package"
      }
    }

    stage('Docker Build Image'){
      steps{
        sh "docker build -t lakshmi131/my-app:1.0.0 ."
      }
    }
    
    stage('push to docker hub'){
      steps{
        withCredentials([string(credentialsId: 'docker-hub', variable: 'dockerPwd')]) {
          sh "docker login -u lakshmi131 -p ${dockerPwd}"
          sh "docker push lakshmi131/myapp:${getLatestCommitId()}"
        }
        
      }
    }

    stage('dev-deploy'){
      steps{
        sshagent(['docker-dev']) {
            sh "ssh -o StrictHostKeyChecking=no ec2-user@172.31.40.194 docker rm -f myweb"
            sh "ssh -o StrictHostKeyChecking=no ec2-user@172.31.40.194 docker run -d -p 8080:8080 --name myweb lakshmi131/myapp:${getLatestCommitId()}"
        }
      }
    }
  }
}
def getLatestCommitId(){
  def commitId = sh returnStdout: true, script: 'git rev-parse --short HEAD'
  return commitId
}
