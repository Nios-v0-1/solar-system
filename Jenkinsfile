pipeline {
	agent any
	
	tools {
		  nodejs 'nodejd-22-12-0'
}

	stages {
		stage('VM Node Version') {

			steps {
				sh "echo 'it should work'"
				sh '''
					node -v
					npm -v
				  '''
			}

		}

	}

}



