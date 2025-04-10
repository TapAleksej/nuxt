def GIT_URL = "https://github.com/AnastasiyaGapochkina01/nuxt.git"
def remote = [:]

pipeline {
  agent {
    label 'yandex'
  }
  
  parameters {
    gitParameter (name: 'branch', type: 'PT_BRANCH', quickFilterEnabled: true)
  }
  
  environment {
    HOST = "10.129.0.25"
    PRJ_DIR = "/var/www/html"
  }
  
  stages {
    stage('Checkout SCM') {
      steps {
        checkout([$class: 'GitSCM', branches: [[name: "${branch}"]], doGenerateSubmoduleConfigurations: false, extentions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'jenkins-ssh-key', url: "$GIT_URL"]]])
      }
    }
    stage('Build static') {
      agent {
        docker {
          image 'node'
          label 'yandex'
          reuseNode true
        }
      }
      steps {
        sh "npm ci --cache .npm --prefer-offline"
        sh "npm run generate"
      }
    }
    stage('Delivery') {
      steps {
        withCredentials([sshUserPrivateKey(credentialsId: 'jenkins-yandex-agent-1-key', keyFileVariable: 'private_key', usernameVariable: 'username')]) {
          script {
            remote.name = "${env.HOST}"
            remote.host = "${env.HOST}"
            remote.user = "$username"
            remote.identity = readFile("$private_key")
            remote.allowAnyHosts = true
            remote.agentForwarding = true
          }
        }
      
        sshCommand remote: remote, command: """
          sudo chown jenkins "${env.PRJ_DIR}"
        """
        sh """
          rsync -av --delete -e "ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -i '${private_key}'" .output/public "jenkins@${env.HOST}:${env.PRJ_DIR}/"
        """
      }
    }
  }
}
