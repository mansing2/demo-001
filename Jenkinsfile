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
	withMaven(maven: 'Maven 3') {
     	dir('app') {
         sh 'mvn clean package'
         dockerCmd "build --tag digitaldemo-docker-snapshot-images.jfrog.io/sparktodo-${JOB_NAME}:SNAPSHOT ."
     	}
	}
    }

    //STAGE3
    stage('Deploy & Test') {
	 dir('app') {
       dockerCmd "run -d -p 9999:9999 --name 'snapshot' --network='host' digitaldemo-docker-snapshot-images.jfrog.io/sparktodo-${JOB_NAME}:SNAPSHOT"
   }

   try{

   echo 'Testing Endpoint'

   sleep(time:10,unit:"SECONDS")
   def get = new URL("http://localhost:9999").openConnection();
   def getRC = get.getResponseCode();
   println(getRC);
   if(getRC.equals(200)) {
     println(get.getInputStream().getText());
   }
   }finally{
     dockerCmd 'rm -f snapshot'
   }
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
