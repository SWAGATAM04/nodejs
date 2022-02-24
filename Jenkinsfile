node {
    
	

    env.AWS_ECR_LOGIN=true
    def newApp
    def registry = 'swagatam04/nodejs-build'
    def registryCredential = 'dockerhub'
	
	stage('Git') {
		git 'https://github.com/SWAGATAM04/nodejs.git'
	}
	stage('Build') {
		sh 'npm install'
	}
	stage('Test') {
		sh 'npm test'
	}
	stage('Building image') {
        docker.withRegistry('https://' + registry, registryCredential) {
		    def buildName = registry + ":$BUILD_NUMBER"
			newApp = docker.build buildName
			newApp.push()
        }
	}
	stage('Registring image') {
		
        docker.withRegistry( 'https://index.docker.io' + registryCredential ) {
    		newApp.push 'latest2'
        }
	}
    stage('Removing image') {
        sh "docker rmi $registry:$BUILD_NUMBER"
        sh "docker rmi $registry:latest"
    }
    
}
