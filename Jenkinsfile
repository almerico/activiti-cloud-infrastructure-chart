pipeline {
    agent {
      label "jenkins-maven" 
    }
    options {
      disableConcurrentBuilds()
    }  
    environment {
      ORG               = 'activiti'
      APP_NAME          = 'application'
      CHARTMUSEUM_CREDS = credentials('jenkins-x-chartmuseum')
      GITHUB_CHARTS_REPO    = "https://github.com/Activiti/activiti-cloud-helm-charts.git"
      GITHUB_HELM_REPO_URL = "https://activiti.github.io/activiti-cloud-helm-charts/"


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
           //check if all versions of activiti-cloud-dependencies are the same
           sh 'make validate'
           dir("./charts/$APP_NAME") {
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

            dir("./charts/$APP_NAME") {
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
          dir("./charts/$APP_NAME") {
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
