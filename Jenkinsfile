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
               git branch: 'sonar', url: 'https://github.com/darinpope/java-web-app.git'
            }
        }
        
	stage('Artifactory Publish') {
            steps {
                script {
		        rtMaven.tool = 'M3'
			rtMaven.deployer snapshotRepo: ARTIFACTORY_LOCAL_SNAPSHOT_REPO, server: server
        		rtMaven.resolver snapshotRepo: ARTIFACTORY_VIRTUAL_SNAPSHOT_REPO, server: server
		        buildInfo = Artifactory.newBuildInfo()
                }
            }
        }
	    
	    
        stage ('Build & Test') {
            steps {
		script {
			rtMaven.run pom: '/var/lib/jenkins/workspace/SDKTech-DevSecOps-Demo/pom.xml', goals: 'clean install', buildInfo: buildInfo
			rtMaven.deployer.deployArtifacts buildInfo
		    	server.publishBuildInfo buildInfo
        	}		
            }
            post {
               success {
                    junit 'target/surefire-reports/**/*.xml'
                }   
            }
        }
	    
	
        
        stage('Security Analysis'){
            steps{
                   withSonarQubeEnv(installationName: 'MySQ') {
                        sh 'mvn clean org.sonarsource.scanner.maven:sonar-maven-plugin:3.9.0.2155:sonar'
                    }
            }
        }
        
        stage('Building Docker Image'){
            steps{
                sh 'sudo docker build -t sdktech-devsecops-demo:$BUILD_NUMBER .'
                sh 'sudo docker images'
            }
        }
        
        stage('Vulnerability Scanning'){
            steps{
               sh 'sudo trivy image sdktech-devsecops-demo:$BUILD_NUMBER > $WORKSPACE/trivy-image-scan-$BUILD_NUMBER.txt'
               
            }
        }
        
        stage('Deploy App'){
            steps{
                sh 'sudo docker run --name SDKTech-DevSecOps-Demo-$BUILD_NUMBER -p 9090:9090 --cpus="0.50" --memory="256m" -e PORT=9090 -d sdktech-devsecops-demo:$BUILD_NUMBER'
            }
        }
        
        stage('Slack it'){
            steps {
                slackSend channel: '#jenkins-status', 
                          message: 'Hello, world'
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
