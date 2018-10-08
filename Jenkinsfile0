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
        withMaven(maven: 'Maven 3') {
            dir('app') {
                sh 'mvn clean package'
                dockerCmd "build --tag digitaldemo-docker-snapshot-images.jfrog.io/sparktodo-${JOB_NAME}:SNAPSHOT ."
            }
        }
    }

    stage('Deploy') {
            dir('app') {
                dockerCmd "run -d -p 9999:9999 --name 'snapshot' --network='host' digitaldemo-docker-snapshot-images.jfrog.io/sparktodo-${JOB_NAME}:SNAPSHOT"
            }
    }

    stage('Tests') {

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


    stage('Push Snapshots to Artifactory'){

       def server = Artifactory.server('abhaya-docker-artifactory')
       def uploadSpec = """{
	        "files": [
		        {
		            "pattern": "**/*.jar",
		              "target": "ext-snapshot-local/"
		              }
	                 ]
	                  }"""
	    server.upload(uploadSpec)

       // Create an Artifactory Docker instance. The instance stores the Artifactory credentials and the Docker daemon host address:
       def rtDocker = Artifactory.docker server: server, host: "tcp://localhost:2375"

       // Push a docker image to Artifactory (here we're pushing hello-world:latest). The push method also expects
       // Artifactory repository name (<target-artifactory-repository>).
       def buildInfo = rtDocker.push "digitaldemo-docker-snapshot-images.jfrog.io/sparktodo-${JOB_NAME}:SNAPSHOT", 'docker-snapshot-images'

       //Publish the build-info to Artifactory:
       server.publishBuildInfo buildInfo

    }
	  stage('Wait for Approval'){
		  input 'Release project for Deployment?'
	  }
    stage('Release') {
        withMaven(maven: 'Maven 3') {
            dir('app') {
                releasedVersion = getReleasedVersion()
                withCredentials([usernamePassword(credentialsId: 'github-cred', passwordVariable: 'password', usernameVariable: 'username')]) {
                    sh "git config user.email test@digitaldemo-docker-release-images.jfrog.io.com && git config user.name Jenkins"
                    sh "mvn release:prepare release:perform -Dusername=${username} -Dpassword=${password}"
                }
                dockerCmd "build --tag digitaldemo-docker-release-images.jfrog.io/sparktodo-${JOB_NAME}:${releasedVersion} ."
            }
        }
    }

    stage('Push image and Artifact Releases to Artifactory'){
       // Create an Artifactory server instance:
       def server = Artifactory.server('abhaya-docker-artifactory')
       def uploadSpec = """{
	"files": [
		{
		"pattern": "**/*.jar",
		"target": "ext-release-local/"
		}
	]
	}"""
	server.upload(uploadSpec)


       // Create an Artifactory Docker instance. The instance stores the Artifactory credentials and the Docker daemon host address:
       def rtDocker = Artifactory.docker server: server, host: "tcp://localhost:2375"

       // Push a docker image to Artifactory (here we're pushing hello-world:latest). The push method also expects
       // Artifactory repository name (<target-artifactory-repository>).
       def buildInfo = rtDocker.push "digitaldemo-docker-release-images.jfrog.io/sparktodo-${JOB_NAME}:${releasedVersion}", 'docker-release-images'

       //Publish the build-info to Artifactory:
       server.publishBuildInfo buildInfo

    }

    stage('Deploy @ Prod') {
        dockerCmd "run -d -p 9999:9999 --name 'production' digitaldemo-docker-release-images.jfrog.io/sparktodo-${JOB_NAME}:${releasedVersion}"
    }
  }
}

def dockerCmd(args) {
    sh "sudo ${DOCKER}/docker ${args}"
}

def getReleasedVersion() {
    return (readFile('pom.xml') =~ '<version>(.+)-SNAPSHOT</version>')[0][1]
}
