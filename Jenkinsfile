pipeline {
    agent any
    tools {
        maven 'maven-3'
        }
    stages{
        stage('GIT checkout') {
            steps {
                git credentialsId: 'git_creds', url: 'https://github.com/Rajtestgit/CalTech-PG-DevOps-Final-Capstone-Project.git'
           }
       }
          stage('Build Package') {
            steps {
                sh 'mvn clean install'
            }
        }
        stage('Docker build and Tag') {
            steps{
                sh 'docker build -t ${JOB_NAME}:v1.${BUILD_NUMBER} .'
                sh 'docker tag ${JOB_NAME}:v1.${BUILD_NUMBER} rajendocker/${JOB_NAME}:v1.${BUILD_NUMBER} '
                        }
        }
        stage('push conatiner') {
            steps{
                withCredentials([string(credentialsId: 'docker-pwd', variable: 'dockerHubPwd')]) {
                  sh "docker login -u rajendocker -p ${dockerHubPwd}"
                  sh 'docker push rajendocker/${JOB_NAME}:v1.${BUILD_NUMBER}'
                } 
                stage('Delete Old Container'){
	                sshagent (credentials: ['dev-server']) {
	             try{
		         sh 'dockrRm = "docker rm -f docker-app'
			     sh 'dockrRmImage = docker rmi  rajendocker/${JOB_NAME}:v1.${BUILD_NUMBER}'
	             sh 'ssh -o StrictHostKeyChecking=no ec2-user@172.31.0.227  ${dockrRm}'
			     sh 'ssh -o StrictHostKeyChecking=no ec2-user@172.31.0.227  ${dockrRmImage}'
		         }catch(e){
			  echo "container docker-app not found" 
            }
           
        }
		}
        stage("Docker Deploy Dev"){
            steps{
                 sshagent (credentials: ['dev-server']){
                     sh 'docker run -p 8080:8080 -d --name docker-app rajendocker/${JOB_NAME}:v1.${BUILD_NUMBER}'
                     sh 'ssh -o StrictHostKeyChecking=no ec2-user@172.31.0.227 ${docker-app}'
                    }
        }          
    }
}
}
   
