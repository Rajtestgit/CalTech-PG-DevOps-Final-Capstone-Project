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
                  sh 'docker push rajendocker/${JOB_NAME}/v1.${BUILD_NUMBER}'
                }      
            }
        }
        stage('Docker Deploy') {
            steps{
                ansiblePlaybook credentialsId: 'ansible-host', disableHostKeyChecking: true, installation: 'ansible', inventory: 'inventory.txt', playbook: 'deploy.yml'
            }
        }          
    }
}
