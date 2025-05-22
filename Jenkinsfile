@Library("PSL@LKG")
@Library('fusion-pipeline-configuration')
@Library('fusion-psl@marason/debug-github')

import groovy.transform.Field
import groovy.json.JsonBuilder
import groovy.json.JsonOutput
import groovy.json.JsonSlurperClassic

Globals.debug = true
Globals.urls.github = "https://github.com"
configureJob(preset: "none")

initReporting(
  slack: [
    includeCommitList: buildInfo.branch.isPr
  ]
)

reportBuild()

pipeline {
  agent none
  options {
    timeout(time: 1, unit: 'HOURS')
    timestamps()

    skipDefaultCheckout(true)
    ansiColor('xterm')
  }

  stages {
    stage("Pipeline") {
      parallel {
        stage ('CI') {
          agent {
            kubernetes {
              label pipelineConfig.data.kubernetes.standard.label
              cloud pipelineConfig.data.kubernetes.standard.cloud
              defaultContainer pipelineConfig.data.kubernetes.standard.defaultContainer
              inheritFrom pipelineConfig.data.kubernetes.standard.inheritFrom
              yaml pipelineConfig.data.kubernetes.standard.template
              retries 2
            }
          }
          environment {
            STAGE_REPORTED = "${STAGE_NAME}"
          }
          stages {
            stage ('Checkout') {
              steps {
                sh "echo hi"
                reportCompositeStage()
                withStageReport {
                  runStandardStage()
                }
              }
            }
          }
          post {
            always {
              reportCompositeStage(final: true)
            }
          }
        }
      }
    }
  }

  post {
    always {
      reportBuild(final: true)
    }
  }
}
