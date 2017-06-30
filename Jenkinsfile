def DOCKER_IMG_BASENAME='demo-app'
def GIT_SHORT_CHANGESET='latest'

node('maven-jdk-8') {
  stage('Checkout code') {
    checkout scm
    GIT_SHORT_CHANGESET = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
  }
    stage('Build') {
      sh "mvn package"
      junit '**/target/surefire-reports/*.xml'
      stash includes: '**/target/*.jar', name: 'application-binaries'
    }
    stage('Integration Tests') {
      sh "mvn verify -fn"
      junit '**/target/failsafe-reports/*.xml'
    }
}
node('docker') {
  stage('Docker Build') {
    checkout scm
    unstash 'application-binaries'
    sh "docker build -t ${DOCKER_IMG_BASENAME}:${GIT_SHORT_CHANGESET} ./"
  }
}
// Putting "input" inside a stage will allow the GUI to visualize it and show the popup
// If not you will have to go to the Console Output of the build
stage('Waiting Approval') {
  // We do not put "input" within a node to avoid eating an executor while waiting
  timeout(time:1, unit:'DAYS') {
    input message: "Is it ok to deploy docker image?", ok: 'Deploy'
  }
}

// We do not put "build" within a node for same reasons as "input"
stage('Deploy') {
  // This will call the downstream job and block until it finishes
  // Unless disabled, downstream job status will be reported to the pipeline
  build job: 'demoapp-staging-deployer',
    parameters: [string(name: 'DOCKER_IMAGE',
    value: "${DOCKER_IMG_BASENAME}:${GIT_SHORT_CHANGESET}")]
}
