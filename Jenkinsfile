pipeline {
    agent any
    tools { 
        maven 'M3' 
    }
    stages {
        stage('Code Checkout') {
            steps {
               git branch: 'sonar', url: 'https://github.com/darinpope/java-web-app'
            }
        }
        
        stage ('Build & Test') {
            steps {
                sh 'mvn install' 
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
    }
  }
        
