pipeline {
    options {
      disableConcurrentBuilds()
    }  
    agent {
      label "jenkins-maven-java11"
    }
    environment {
      ORG               = 'activiti'
      APP_NAME          = 'activiti-cloud-full-example'

      GITHUB_CHARTS_REPO    = "https://github.com/Activiti/activiti-cloud-helm-charts.git"
      GITHUB_HELM_REPO_URL = "https://activiti.github.io/activiti-cloud-helm-charts/"
      HELM_RELEASE_NAME = "example-$BRANCH_NAME-$BUILD_NUMBER".toLowerCase()

      PREVIEW_VERSION = "0.0.0-SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER"
      PREVIEW_NAMESPACE = "example-$BRANCH_NAME-$BUILD_NUMBER".toLowerCase()
      GLOBAL_GATEWAY_DOMAIN="35.228.195.195.nip.io"
      REALM = "activiti"
    }
    stages {
      stage('CI Build and push snapshot') {
        when {
          branch 'PR-*'
        }
        environment {
         GATEWAY_HOST = "activiti-cloud-gateway.$PREVIEW_NAMESPACE-security.$GLOBAL_GATEWAY_DOMAIN"
         SSO_HOST = "activiti-cloud-gateway.$PREVIEW_NAMESPACE-security.$GLOBAL_GATEWAY_DOMAIN"
        }
        steps {
          container('maven') {
          sh 'make validate'
           dir ("./charts/$APP_NAME") {
	           // sh 'make build'
              sh 'make install-security'
            }

            dir("./activiti-cloud-acceptance-scenarios") {
              git 'https://github.com/Activiti/activiti-cloud-acceptance-scenarios.git'
              sh "mvn clean install -DskipTests && mvn -pl 'security-policies-acceptance-tests' clean verify"
            }
          }
        }
      }
      stage('Build Release') {
        when {
          branch 'master'
        }
	environment {
         GATEWAY_HOST = "activiti-cloud-gateway.$PREVIEW_NAMESPACE-security.$GLOBAL_GATEWAY_DOMAIN"
         SSO_HOST = "activiti-cloud-gateway.$PREVIEW_NAMESPACE-security.$GLOBAL_GATEWAY_DOMAIN"
        }      
        steps {
          container('maven') {
            // ensure we're not on a detached head
            sh "git checkout master"
            sh "git config --global credential.helper store"

            sh "jx step git credentials"
            // so we can retrieve the version in later steps
            sh "echo \$(jx-release-version) > VERSION"
        //RUNTIME bundle tests
            dir ("./charts/$APP_NAME") {
	           // sh 'make build'
              sh 'make install-security'
            }

            dir("./activiti-cloud-acceptance-scenarios") {
              git 'https://github.com/Activiti/activiti-cloud-acceptance-scenarios.git'
              sh "mvn clean install -DskipTests && mvn -pl 'security-policies-acceptance-tests' clean verify"
            }	       
            }
          }
        }
      
    }
   post {
	failure {
           slackSend(
             channel: "#activiti-community-builds",
             color: "danger",
             message: "activiti-cloud-full-chart-security is failed $BUILD_URL"
           )
        }    
        always {
          container('maven') {
            dir("./charts/$APP_NAME") {
               sh "make delete-security"
            }
            sh "kubectl delete namespace $PREVIEW_NAMESPACE-security" 
          }
          cleanWs()
        }
   }
}
