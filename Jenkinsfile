def options = '-S'
def gradle = "./gradlew ${options}"

pipeline {
   agent { label 'gce-obi-12.2.1.4' }

   environment {
      ORG_GRADLE_PROJECT_githubToken = credentials('github-redpillanalyticsbot-secret')
		AWS = credentials("rpa-development-build-server-svc")
		AWS_ACCESS_KEY_ID = "${env.AWS_USR}"
		AWS_SECRET_ACCESS_KEY = "${env.AWS_PSW}"
		GRADLE_COMBINED = credentials("gradle-publish-key")
		GRADLE_KEY = "${env.GRADLE_COMBINED_USR}"
		GRADLE_SECRET = "${env.GRADLE_COMBINED_PSW}"
      ADMIN = credentials('obiee-admin-user')
      repositoryPassword = credentials('obiee-repository-password')
      JENKINS_NODE_COOKIE = 'dontKillMe'
   }

   stages {
      stage('Assemble') {
         when { changeRequest() }
         parallel {
            stage('Startup') {
               steps {
                  sh "$gradle startServices"
               }
            }
            stage('Build') {
               steps {
                  sh "$gradle featureCompare buildZip deployZip"
               }
            }
         }
         post {
            always {
               archiveArtifacts artifacts: 'obi/build/distributions/*.zip', fingerprint: true, allowEmptyArchive: true
               sh "$gradle producer"
            }
         }
      }

      stage('Baseline Test') {
         when { changeRequest() }
         steps {
            sh "$gradle featureBaselineWorkflow"
         }
         post {
            always {
               junit testResults: 'obi/build/test-groups/**/*.xml', allowEmptyResults: true
               sh "$gradle producer"
            }
         }
      }

      stage('Revision Test') {
         when { changeRequest() }
         steps {
            sh "$gradle revisionWorkflow"
         }
         post {
            always {
               junit testResults: 'obi/build/test-groups/**/*.xml', allowEmptyResults: true
               sh "$gradle producer"
            }
         }
      }

      stage('Publish') {
         when { branch "master" }
         steps {
            sh "$gradle featureCompare publish"
         }
         post {
            always {
               archiveArtifacts artifacts: 'obi/build/distributions/*.zip', fingerprint: true, allowEmptyArchive: true
               sh "$gradle producer"
            }
         }
      }

      stage('Deploy to QA') {
         when { branch "master" }
         steps {
            sh "$gradle importWorkflow"
         }
         post {
            always {
               sh "$gradle producer"
            }
         }
      }
   }
}