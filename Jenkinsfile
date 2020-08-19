def options = '-S'
def properties = "-PbuildId=${env.BUILD_TAG}"
def checkmate = "./gradlew ${options} ${properties}"

pipeline {
  agent {
    label 'amzn-obi-12.2.1.4'
  }
  environment {
    // Jenkins credentials used to store sensitive information
    ADMIN = credentials('adminUser')
    repositoryPassword = credentials('repositoryPassword')
  }
  stages {

    stage('Build and Startup') {

      steps {
        // start up OBIEE while also doing the build
        parallel (
          "OBI Startup": {
            sh "${checkmate} startOBI"
          },
	
          // build with patches
          "Build": {
            sh "${checkmate} cleanJunit featureCompare"
          }
	
	      ) // end of parallel function
      }
    } // end of Build and Startup stage

    stage('Test') {
         when { changeRequest() }
         environment {
            JENKINS_NODE_COOKIE = 'dontKillMe'
         }
      steps{
        // run baseline and revision, and then register the results
        sh "${checkmate} featureBaselineWorkflow"
        sh "${checkmate} featureRevisionWorkflow"
        junit allowEmptyResults: true, testResults: "build/test-groups/*/xml-reports/*.xml"
      }
       post {
          always {
             junit allowEmptyResults: true, testResults: "build/test-groups/*/xml-reports/*.xml"
             archiveArtifacts artifacts: 'build/distributions/*.zip', fingerprint: true, allowEmptyArchive: true
             sh "$checkmate producer"
          }
       }
    }

    stage('New Release') {
      when {
	      branch 'master'
      }
      // create a new release when the build is against master branch
      steps {
	      sh "${checkmate} release -Prelease.disableChecks -Prelease.localOnly"
      }
    } // end of New Release stage

    // Publish distribution files to Maven
    stage('Publish Artifacts') {
      steps {
	      sh "${checkmate} -p ${projectDir} publish"
      }
    } // end of Publish Artifacts stage

    stage('Publish Release') {

      when {
	      branch 'master'
      }

      // publish new releases to GitHub as well
      // This creates a new Github release and attaches distribution files
      steps {
	      sh "${checkmate} githubRelease"
      }
    } // end of New Release stage
    
  } // end of Stages block

  post {
    always {
      archiveArtifacts artifacts: 'build/distributions/*.zip', fingerprint: true, allowEmptyArchive: true
      sh "${checkmate} producer"
    }
  }
}
