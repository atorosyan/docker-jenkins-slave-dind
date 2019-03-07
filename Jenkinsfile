import java.text.SimpleDateFormat

pipeline {
  agent any
  
  options {
    buildDiscarder(logRotator(numToKeepStr: '2'))
    disableConcurrentBuilds()
  }
  stages {
    stage("build") {
      steps {
        script {
          def dateFormat = new SimpleDateFormat("yy.MM.dd")
          currentBuild.displayName = dateFormat.format(new Date()) + "-" + env.BUILD_NUMBER
        }
        sh "docker image build -t artashes/jenkins-swarm-agent ."
      }
    }
    stage("release") {
      when {
        branch "master"
      }
      steps {
//        dockerLogin()
        withRegistry("http://192.168.99.105:5000") {
          sh "docker tag artashes/jenkins-swarm-agent artashes/jenkins-swarm-agent:${currentBuild.displayName}"
          sh "docker image push artashes/jenkins-swarm-agent:latest"
          sh "docker image push artashes/jenkins-swarm-agent:${currentBuild.displayName}"
        }
      }
    }
    stage("deploy") {
      when {
        branch "master"
      }
      agent {
        label "prod"
      }
      steps {
        dfDeploy("jenkins-swarm-agent", "infra_jenkins-agent", "")
      }
    }
  }
  post {
    success {
      slackSend(
        color: "good",
        message: "artashes/jenkins-swarm-agent:${currentBuild.displayName} was deployed to the cluster. Verify that it works correctly!"
      )
    }
    failure {
      slackSend(
        color: "danger",
        message: "${env.JOB_NAME} failed: ${env.RUN_DISPLAY_URL}"
      )
    }
  }
}
