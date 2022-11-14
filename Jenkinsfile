def server = Artifactory.server 'MyJFrogServer'
def rtMaven = Artifactory.newMavenBuild()
def buildInfo
def ARTIFACTORY_LOCAL_SNAPSHOT_REPO = 'sdk-demo-webapp-libs-snapshot-local/'
def ARTIFACTORY_VIRTUAL_SNAPSHOT_REPO = 'sdk-demo-webapp-libs-snapshot-local/'


pipeline {
    agent any
    tools { 
        maven 'M3'
    }
    options { 
    timestamps () 
    buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '10', numToKeepStr: '5')	
    }
    stages {
        stage('Code Checkout') {
            steps {
               git branch: 'sonar', url: 'https://github.com/mohanparsha/SDKTech-DevOps-Demo.git'
            }
        }
   	    
        stage ('Build & Test') {
            steps {
		script {
			rtMaven.tool = 'M3'
			rtMaven.deployer snapshotRepo: ARTIFACTORY_LOCAL_SNAPSHOT_REPO, server: server
        		buildInfo = Artifactory.newBuildInfo()
			rtMaven.run pom: '/var/lib/jenkins/workspace/SDKTech-DevSecOps-Demo/pom.xml', goals: 'clean install', buildInfo: buildInfo
        	}		
            }
            post {
               success {
                    junit 'target/surefire-reports/**/*.xml'
                }   
            }
        }
	    
	stage('Publish Artifact') {
            steps {
                script {
		        rtMaven.deployer.deployArtifacts buildInfo
		    	server.publishBuildInfo buildInfo
                }
            }
        }
           
        stage('Building Docker Image'){
            steps{
		sh 'sudo chmod +x /var/lib/jenkins/workspace/SDKTech-DevSecOps-Demo/mvnw'
                sh 'sudo docker build -t sdktech-devsecops-demo:$BUILD_NUMBER .'
                sh 'sudo docker images'
            }
        }
	
        stage('QA Release'){
            steps{
                sh 'sudo docker run --name SDKTech-DevSecOps-Demo-$BUILD_NUMBER -p 9090:9090 --cpus="0.50" --memory="256m" -e PORT=9090 -d sdktech-devsecops-demo:$BUILD_NUMBER'
            }
        }
	
	stage ("UAT Approval") {
            steps {
                script {
			mail from: "mohan.parsha@gmail.com", to: "mohan.parsha@gmail.com", subject: "APPROVAL REQUIRED FOR UAT Release - $JOB_NAME" , body: """Build $BUILD_NUMBER required an approval. Go to $BUILD_URL for more info."""
                    	def deploymentDelay = input id: 'Deploy', message: 'Release to UAT?', parameters: [choice(choices: ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11', '12', '13', '14', '15', '16', '17', '18', '19', '20', '21', '22', '23', '24'], description: 'Hours to delay deployment?', name: 'deploymentDelay')]
        	    	sleep time: deploymentDelay.toInteger(), unit: 'HOURS'
                }
            }
        }
	    
	stage('UAT Release'){
            steps{
                sh 'echo Build Released to UAT'
            }
        }
	    
	stage ("PROD Approval") {
            steps {
                script {
			mail from: "mohan.parsha@gmail.com", to: "mohan.parsha@gmail.com", subject: "APPROVAL REQUIRED FOR PROD Release - $JOB_NAME" , body: """Build $BUILD_NUMBER required an approval. Go to $BUILD_URL for more info."""
                    	def deploymentDelay = input id: 'Deploy', message: 'Release to UAT?', parameters: [choice(choices: ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11', '12', '13', '14', '15', '16', '17', '18', '19', '20', '21', '22', '23', '24'], description: 'Hours to delay deployment?', name: 'deploymentDelay')]
        	    	sleep time: deploymentDelay.toInteger(), unit: 'HOURS'
                }
            }
        }
	    
	stage('PROD Release'){
            steps{
                sh 'echo Build Released to PROD'
            }
        }
    }
    post {
    	always {
		mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br>URL: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "Success: Project name -> ${env.JOB_NAME}", to: "mohan.parsha@gmail.com";
    	}
    	failure {
      		sh 'echo "This will run only if failed"'
      		mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br>URL: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "ERROR: Project name -> ${env.JOB_NAME}", to: "mohan.parsha@gmail.com";
    	}
  }
}
