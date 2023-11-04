pipeline {

  agent { label 'kube' }

  stages {

    stage('Checkout Source') {
      steps {
        git url:'https://github.com/akshaybaluapuri/simple-nginx-deployment.git', branch:'main'
      }
    }

    stage('Deploy App') {
      steps {
        script {
          kubernetesDeploy(configs: "nginx-deployment.yaml", kubeconfigId: "mykubeconfig")
        }
      }
    }

  }

}
