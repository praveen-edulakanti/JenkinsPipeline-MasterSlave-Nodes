pipeline {
      agent none
      stages {
		    stage('AnsibleServer'){
            agent {
                label "ansiblejob"
            } 
            stages {
               stage('CleanWorkspace') {
                    steps {
                        sh 'echo ${WORKSPACE}'
                        sh 'echo ${BUILD_NUMBER}'
                        cleanWs()
                    }
               }
               stage('Code Pull'){
                 steps {
                    git url: 'https://github.com/praveen-edulakanti/Ansible-PHPApplication-Deployment'
                  }
               }
               stage('Run PlayBook on Ansible Slave') {
                    steps {
                        sh 'ansible-playbook -i inventory ansible-playbook.yml'
                        sh 'ls -l'
                        sh 'pwd'
                    }
               }    
	         }       
      }
 		 stage('DockerServer') {
            agent {
                label 'dockerjob'
            }
			environment{
				DOCKER_URL = "praveenedulakanti"
				IMAGE_URL_WITH_TAG = "${DOCKER_URL}/phpapp-devops:${BUILD_NUMBER}"
				DOCKER_USER_NAME = "praveenedulakanti"
				APP_NAME = "appdeploy"
		    }
		 stages {
            stage('CleanWorkspace') {
                steps {
                    sh 'echo ${WORKSPACE}'
                    sh 'echo ${BUILD_NUMBER}'
                    cleanWs()
                }
            }
            stage('Code Pull'){
                steps {
                    git url: 'https://github.com/praveen-edulakanti/Kubernetes-PHP-Mysql-Application.git'
                }
            }
            stage('Build Docker Image') {
                steps {
                    sh 'docker build -t $IMAGE_URL_WITH_TAG .'
                }
            }
	          stage('Push Docker Image'){
 	             steps { 
		             withCredentials([string(credentialsId: 'DOCKER_HUB_PASSWORD', variable: 'DOCKER_HUB_PASSWORD')]) {
                     sh '''
                    	docker login --username=$DOCKER_USER_NAME --password=$DOCKER_HUB_PASSWORD
                 	    docker push $IMAGE_URL_WITH_TAG
	                   '''      
         	       }
	             }
		        }
		    stage('Docker Image Stop & Remove'){
     	        when {
                expression {
                    def containerId = sh(script: 'docker ps -f name=$APP_NAME -q', returnStdout: true)
                    return containerId.trim() != ""
                 }
		          }
		         steps {
                sh '''
       	            docker stop $APP_NAME 
		                docker rm -f $APP_NAME
		               '''
                 }
             }
		    stage('Docker Image Run'){
     	     steps { 
                sh 'docker run  -d  -p 8081:80  --name $APP_NAME $IMAGE_URL_WITH_TAG'
              }
         }
			}	
				
		}	
	}
}	
