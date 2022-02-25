pipeline { 
    environment { 
        AWS_ACCOUNT_ID="295317182281"
        AWS_DEFAULT_REGION="us-east-2" 
        IMAGE_REPO_NAME="nodejs-build"
        IMAGE_TAG=""
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
        dockerImage = '' 
    }
    agent any 
      stages { 
	stage('Git') {
		steps {
			git 'https://github.com/SWAGATAM04/nodejs.git'
	           }
	}
	stage('Build') {
		steps {
			sh 'npm install'
	              }
	}
	stage('Test') {
	        steps {
			sh 'npm test'
		      }
	}
	
	stage('Logging into AWS ECR') {
            steps {
                script {
                sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                }
                 
            }
        }
        
	stage('Building our image') { 
            steps { 
                script { 
			dockerImage = docker.build "${IMAGE_REPO_NAME}":"${BUILD_NUMBER}" 
                }
            } 
        }
        stage('Pushing to ECR') { 
            steps { 
                script { 
			sh "docker tag ${IMAGE_REPO_NAME}:${BUILD_NUMBER} ${REPOSITORY_URI}:${BUILD_NUMBER}"
                        sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${BUILD_NUMBER}"
                   }
                } 
            }
         
        stage('Cleaning up') { 
            steps { 
		    sh "docker rmi ${IMAGE_REPO_NAME}:$BUILD_NUMBER" 
        } 
      }
   }
}	
    
