void setBuildStatus(String message, String state, String repo_url) {
  step([
      $class: "GitHubCommitStatusSetter",
      reposSource: [$class: "ManuallyEnteredRepositorySource", url: repo_url],
      contextSource: [$class: "ManuallyEnteredCommitContextSource", context: "ci/jenkins/build-status"],
      errorHandlers: [[$class: "ChangingBuildStatusErrorHandler", result: "UNSTABLE"]],
      statusResultSource: [ $class: "ConditionalStatusResultSource", results: [[$class: "AnyBuildResult", message: message, state: state]] ]
  ]);
}

pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/FortinetCloudCSE/oci-solutions'
    }

    stages {

       stage('Checkout') {
            steps {
              script {
                // Capture SCM vars explicitly; GIT_URL is only injected into env
                // by the git plugin's own checkout, which lightweight/API-only
                // Jenkinsfile fetches skip entirely.
                def scmVars = checkout scm
                env.REPO_URL = scmVars.GIT_URL ?: env.REPO_URL
              }
            }
       }

       stage('Checking for question/discussion section in content folders'){
            when { expression { false } }
            steps {
              script {
                def warningFound = false
                try {
                  sh '''
                  for dir in content/*/; do
                    found=false
                    for file in "$dir"/*; do
                      if grep -qiE 'discussion|questions|q&a' "$file" >/dev/null 2>&1; then
                        found=true
                        break
                      fi
                    done
                    if [ "$found" == "true" ]; then
                      echo "$dir: relevant section found"
                    else
                      echo "$dir: sections not found"
                      warningFound=true
                    fi
                  done
                  if [ "$warningFound" == "true" ]; then
                    echo "Warning: some content directories did not contain any files with a discussion/questions/q&a section."
                  fi
                  '''
                } catch (Exception e) {
                   echo "An error occurred: ${e.message}"
                }
              }
            }
       }
    }
    post {
     success {
        setBuildStatus("Build succeeded", "SUCCESS", env.REPO_URL);
     }
     failure {
        setBuildStatus("Build failed", "FAILURE", env.REPO_URL);
     }
  }
}
