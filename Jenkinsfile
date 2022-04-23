pipeline {
  agent any
    tools {
      maven 'maven'
      jdk 'openjdk-11'
    }
    environment{
        AWS_ACCOUNT_ID="490382235718"
        AWS_DEFAULT_REGION="ap-south-1" 
        IMAGE_REPO_NAME="devops-1"
        IMAGE_TAG="latest"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"	    
    }
    stages{
        stage('Logging into AWS ECR') {
            steps {
                script {
                sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                }
             }
        }	         
        stage('Build maven') {
            steps { 
                    sh 'pwd'      
                    sh 'mvn  clean install package'
	          }   
            }
        
        stage('Copy Artifact') {
           steps { 
                   sh 'pwd'
		   sh 'cp -r target/*.jar docker'
           }
        }
        stage('Building image') {
           steps{
           script {
           dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}","./docker"
        }
      }
    }
    	stage('Pushing to ECR') {
           steps{  
            script {
                sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"
                sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
         }
        }
      }
	    stage('Deploy App on k8s') {
           steps {
            sshagent(['shhtok8master']) {
         /*   sh "kubectl delete -f java-app-deployment.yaml"
            sh "rm -rf /home/ec2-user/java-app-deployment.yaml" */
            sh "scp -o StrictHostKeyChecking=no java-app-deployment.yaml ec2-user@3.110.195.3:/home/ec2-user"
           
            script {
                try{
                  sh "ssh ec2-user@3.110.195.3 kubectl apply -f ."
                }catch(error){
                    sh "ssh ec2-user@3.110.195.3 kubectl apply -f ."
            }
}
        }

    }
}  
    }
}
