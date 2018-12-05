pipeline {
    agent {
      label "jenkins-maven"
    }
    environment {
      ORG               = 'activiti'
      APP_NAME          = 'infrastructure'
      CHARTMUSEUM_CREDS = credentials('jenkins-x-chartmuseum')
      GITHUB_CHARTS_REPO    = "https://github.com/Activiti/activiti-cloud-helm-charts.git"
      GITHUB_HELM_REPO_URL = "https://github.com/Activiti/activiti-cloud-helm-charts"


    }
    stages {
      stage('CI Build and push snapshot') {
        when {
          branch 'PR-*'
        }
        environment {
          PREVIEW_VERSION = "0.0.0-SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER"
          PREVIEW_NAMESPACE = "$APP_NAME-$BRANCH_NAME".toLowerCase()
          HELM_RELEASE = "$PREVIEW_NAMESPACE".toLowerCase()
        }
        steps {
          container('maven') {
           dir ('./charts/infrastructure') {
	          sh 'make build'
           }
          }
        }
      }
      stage('Build Release') {
        when {
          branch 'master'
        }
        steps {
          container('maven') {
            // ensure we're not on a detached head
            sh "git checkout master"
            sh "git config --global credential.helper store"

            sh "jx step git credentials"
            // so we can retrieve the version in later steps
            sh "echo \$(jx-release-version) > VERSION"

            dir ('./charts/infrastructure') {
                sh 'make tag'
                sh 'make release'
                sh 'make github'
            }
          }
        }
      }

      stage('Promote to Environments') {
        when {
          branch 'master'
        }
        steps {
          dir ('./charts/infrastructure') {
            container('maven') {
              sh 'jx step changelog --version v\$(cat ../../VERSION)'
              sh 'jx step git credentials'
              sh 'cd ../.. && updatebot push-version --kind helm $APP_NAME \$(cat VERSION)'

            }
          }
        }
      }
    }
    post {
        always {
            cleanWs()
        }
    }
}
