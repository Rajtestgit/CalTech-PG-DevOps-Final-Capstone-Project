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
            steps {
                withCredentials([string(credentialsId: 'docker-pwd', variable: 'dockerHubPwd')]) {
                  sh "docker login -u rajendocker -p ${dockerHubPwd}"
                  sh 'docker push rajendocker/${JOB_NAME}:v1.${BUILD_NUMBER}'
                } 
	    }
	}
	   stage('Run Container on Dev Server'){
		   steps {
			   dockerRun = "docker run -p 8080:8080 -d --name my-app rajendocker/${JOB_NAME}:v1.${BUILD_NUMBER}"
		   	sshagent (['dev-server']) { 
			   sh 'ssh -o StrictHostKeyChecking=no  ec2-user@172.31.0.227 ${dockerRun}'
		  	   }
	   }
}
}
}
