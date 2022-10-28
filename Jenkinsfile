def img
pipeline {
    environment {
        registry = "rajendocker/docker-app" //To push an image to Docker Hub, you must first name your local image using your Docker Hub username and the repository name that you created through Docker Hub on the web.
        registryCredential = 'docker-creds'
        dockerImage = ''
    }
}
    agent any
    stages {
        stage('SCM checkout') {
            steps {
               git branch: 'master', credentialsId: 'git-creds', url: 'github.com/Rajtestgit/CalTech-PG-DevOps-Final-Capstone-Project.git' 
            }
        }
    }
	    stage('Build Package') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage ('Stop previous running container'){
            steps{
                sh returnStatus: true, script: 'docker stop $(docker ps -a | grep ${JOB_NAME} | awk \'{print $1}\')'
                sh returnStatus: true, script: 'docker rmi $(docker images | grep ${registry} | awk \'{print $3}\') --force' //this will delete all images
                sh returnStatus: true, script: 'docker rm ${JOB_NAME}'
            }
        }


        stage('Build Image') {
            steps {
                script {
                    img = registry + ":${env.BUILD_ID}"
                    println ("${img}")
                    dockerImage = docker.build("${img}")
                }
            }
        }



        stage('Test - Run Docker Container on Jenkins node') {
           steps {

                sh label: '', script: "docker run -d --name ${JOB_NAME} -p 5000:5000 ${img}"
          }
        }

        stage('Push To DockerHub') {
            steps {
                script {
                    docker.withRegistry( 'https://registry.hub.docker.com ', registryCredential ) {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Deploy to Test Server') {
            steps {
                script {
                    def stopcontainer = "docker stop ${JOB_NAME}"
                    def delcontName = "docker rm ${JOB_NAME}"
                    def delimages = 'docker image prune -a --force'
                    def drun = "docker run -d --name ${JOB_NAME} -p 5000:5000 ${img}"
                    println "${drun}"
                    sshagent(['dev-server']){
                        sh returnStatus: true, script: "ssh -o StrictHostKeyChecking=no ec2-user@172.31.0.227 ${stopcontainer} "
                        sh returnStatus: true, script: "ssh -o StrictHostKeyChecking=no ec2-user@172.31.0.227 ${delcontName}"
                        sh returnStatus: true, script: "ssh -o StrictHostKeyChecking=no ec2-user@172.31.0.227 ${delimages}"

                    // some block
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@172.31.0.227 ${drun}"
                    }
                }
            }
        }

    }
}
