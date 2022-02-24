pipeline { 
    environment { 
        registry = "swagatam04/nodejs-build" 
        registryCredential = 'dockerhub' 
        dockerImage = '' 
    }
    agent any 
      stages { 
	stage('Git') {
		git 'https://github.com/SWAGATAM04/nodejs.git'
	}
	stage('Build') {
		sh 'npm install'
	}
	stage('Test') {
		sh 'npm test'
	}
	stage('Building our image') { 
            steps { 
                script { 
                    dockerImage = docker.build registry + ":$BUILD_NUMBER" 
                }
            } 
        }
        stage('Deploy our image') { 
            steps { 
                script { 
                    docker.withRegistry( '', registryCredential ) { 
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
}
    
