pipeline { 
    environment { 
        AWS_ACCOUNT_ID="295317182281"
        AWS_DEFAULT_REGION="us-east-2" 
        IMAGE_REPO_NAME="nodejs-build"
        IMAGE_TAG=""
        registry = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
        dockerImage = '' 
	registryCredential = 'ecr-cred'
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
	
	
        
	stage('Building our image') { 
            steps { 
                script { 
			dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
            } 
        }
        stage('Pushing to ECR') { 
            steps { 
                script { 
			 docker.withRegistry('https://295317182281.dkr.ecr.us-east-2.amazonaws.com', 'ecr:us-east-2:ecr-cred') {
                         dockerImage.push()

                   }
                } 
            }
	}
        stage('Cleaning up') { 
            steps { 
		    sh "docker rmi $registry:$BUILD_NUMBER" 
        } 
      }
      
	 stage('Deploy Image to ECS'){
		 steps {
			 sh "REGION=us-east-2"
                         sh "REPOSITORY_NAME=nodejs-build"
                         sh "CLUSTER=node-js-app-production-cluster"
                         sh "FAMILY=`sed -n 's/.*"family": "\(.*\)",/\1/p' taskdef.json`
                         sh "NAME=`sed -n 's/.*"name": "\(.*\)",/\1/p' taskdef.json`
                         sh "SERVICE_NAME=${NAME}-service"
			 sh "env"
                         sh "aws configure list"
                         sh "echo $HOME"
                         #Store the repositoryUri as a variable
                         sh "REPOSITORY_URI=`aws ecr describe-repositories --repository-names ${REPOSITORY_NAME} --region ${REGION} | jq .repositories[].repositoryUri | tr -d '"'`"
                         #Replace the build number and respository URI placeholders with the constants above
                        sh "sed -e "s;%BUILD_NUMBER%;${BUILD_NUMBER};g" -e "s;%REPOSITORY_URI%;${REPOSITORY_URI};g" taskdef.json > ${NAME}-v_${BUILD_NUMBER}.json"
                         #Register the task definition in the repository
                         sh "aws ecs register-task-definition --family ${FAMILY} --cli-input-json file://${WORKSPACE}/${NAME}-v_${BUILD_NUMBER}.json --region ${REGION}"
                         sh "SERVICES=`aws ecs describe-services --services ${SERVICE_NAME} --cluster ${CLUSTER} --region ${REGION} | jq .failures[]`"
                         #Get latest revision
                         sh "REVISION=`aws ecs describe-task-definition --task-definition ${NAME} --region ${REGION} | jq .taskDefinition.revision`"
                         #Create or update service
if [ "$SERVICES" == "" ]; then
  echo "entered existing service"
  DESIRED_COUNT=`aws ecs describe-services --services ${SERVICE_NAME} --cluster ${CLUSTER} --region ${REGION} | jq .services[].desiredCount`
  if [ ${DESIRED_COUNT} = "0" ]; then
    DESIRED_COUNT="1"
  fi
  aws ecs update-service --cluster ${CLUSTER} --region ${REGION} --service ${SERVICE_NAME} --task-definition ${FAMILY}:${REVISION} --desired-count ${DESIRED_COUNT}
else
  echo "entered new service"
  aws ecs create-service --service-name ${SERVICE_NAME} --desired-count 1 --task-definition ${FAMILY} --cluster ${CLUSTER} --region ${REGION}
fi

   }
}	
    
      }
}
