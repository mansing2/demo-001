#!groovy

def releasedVersion

node('master') {
  def dockerTool = tool name: 'docker', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
  withEnv(["DOCKER=${dockerTool}/bin"]) {
    stage('Prepare') {
        deleteDir()
        parallel Checkout: {
            checkout scm
        }
    }

    stage('Build') {

    }

    stage('Deploy & Test') {

    }


    stage('Push Snapshots to Artifactory'){

    }
	  stage('Wait for Approval'){

	  }
    stage('Release') {

    }

    stage('Push image and Artifact Releases to Artifactory'){

    }

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
