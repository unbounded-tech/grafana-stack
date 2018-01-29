pipeline {
  agent {
    label "prod"
  }
  options {
    buildDiscarder(logRotator(numToKeepStr: '2'))
    disableConcurrentBuilds()
  }
  stages {
    stage("notify") {
      steps {
        slackSend(
          color: "info",
          message: "${env.JOB_NAME} started: ${env.RUN_DISPLAY_URL}"
        )
      }
    }
    stage("deploy") {
      when {
        branch "master"
      }
      environment {
        GRAFANA_DOMAIN = "${env.grafanaDomain}"
      }
      steps {
        script {
          if (env.grafanaDomain) {
            withCredentials([usernamePassword(
              credentialsId: "grafana-auth",
              usernameVariable: "GF_USER",
              passwordVariable: "GF_PASSWORD"
            )]) {
              sh "docker stack deploy -c stack.yml grafana"
            }
          } else {
            sh 'echo "ERROR: env.grafanaDomain is required." && exit 1'
          }
        }
      }
    }
  }
  post {
    always {
      sh "docker system prune -f"
    }
    failure {
      slackSend(
        color: "danger",
        message: "${env.JOB_NAME} failed: ${env.RUN_DISPLAY_URL}"
      )
    }
    success {
      slackSend(
        color: "success",
        message: "${env.JOB_NAME} succeeded: ${env.RUN_DISPLAY_URL}"
      )
    }
  }
}