#!groovy

def releasedVersion

node('master') {
  def dockerTool = tool name: 'docker', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
  withEnv(["DOCKER=${dockerTool}/bin"]) {

    //STAGE 1
    stage('Prepare') {
        deleteDir()
        parallel Checkout: {
            checkout scm
        }
    }

    //STAGE2:
    stage('Build') {

    }

    //STAGE3
    stage('Deploy & Test') {

    }

    //STAGE4
    stage('Push Snapshots to Artifactory'){

    }
    //STAGE5
	  stage('Wait for Approval'){

	  }
    //STAGE6
    stage('Release') {

    }
    //STAGE7
    stage('Push image and Artifact Releases to Artifactory'){

    }
    //STAGE8
    stage('Deploy @ Prod') {

    }
  }
}

def dockerCmd(args) {
    sh "sudo ${DOCKER}/docker ${args}"
}

def getReleasedVersion() {
    return (readFile('pom.xml') =~ '<version>(.+)-SNAPSHOT</version>')[0][1]
}
