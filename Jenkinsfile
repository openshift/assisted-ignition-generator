String cron_string = BRANCH_NAME == "master" ? "@daily" : ""

pipeline {
  options { timeout(time: 1, unit: 'HOURS') }
  triggers { cron(cron_string) }
  environment {
            GENERATOR = 'quay.io/ocpmetal/assisted-ignition-generator'
            SLACK_TOKEN = credentials('slack-token')
  }
  agent {
    node {
      label 'host'
    }

  }
  stages {
    stage('build') {
      steps {
        sh 'docker image prune -a -f'
        sh 'make build-image'
      }
    }


  stage('publish images on push to master') {
                when {
                    branch 'master'
                }

                steps {

                    withCredentials([usernamePassword(credentialsId: 'ocpmetal_cred', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh '''docker login quay.io -u $USER -p $PASS'''
                    }

                    sh '''docker tag  ${GENERATOR} ${GENERATOR}:latest'''
                    sh '''docker tag  ${GENERATOR} ${GENERATOR}:${GIT_COMMIT}'''
                    sh '''docker push ${GENERATOR}:latest'''
                    sh '''docker push ${GENERATOR}:${GIT_COMMIT}'''
                }

     }
  }
  post {
    always {
        script {
            if ((env.BRANCH_NAME == 'master') && (currentBuild.currentResult == "ABORTED" || currentBuild.currentResult == "FAILURE")){
                stage('notify master branch fail') {
                        script {
                            def data = [text: "Attention! assisted-ignition-generator branch  test failed, see: ${BUILD_URL}"]
                            writeJSON(file: 'data.txt', json: data, pretty: 4)
                        }
                        sh '''curl -X POST -H 'Content-type: application/json' --data-binary "@data.txt"  https://hooks.slack.com/services/${SLACK_TOKEN}'''
                }
            }
        }
    }
  }
}