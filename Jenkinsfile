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

pipeline {
    agent any

    stages {
      stage('Checkout Source') {
        steps {
          git url:'https://github.com/akshaybaluapuri/simple-nginx-deployment.git', branch:'main'
        }
      }

      stage('Deploy to Kubernetes') {
        steps {
          script {
            def kubeconfig = credentials('np-omnenest-kubernetes')
              kubernetesDeploy(
              credentialsId: 'np-omnenest-kubernetes', // Use the credentials ID
              kubeconfigFile: kubeconfig, // Use the secret file as the kubeconfig
              manifestsPattern: 'nginx-deployment.yaml' // Path to your manifest
                    )
                }
            }
        }
    }
}

