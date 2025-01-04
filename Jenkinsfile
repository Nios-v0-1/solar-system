// edited ...3210
pipeline {
	agent any
	
	tools {
		  nodejs 'nodejd-22-12-0'
		}

	stages {
		stage('Installing Dependencies') {
			steps {
				sh "npm install --no-audit"
				}
		}
		stage('Dependency Scanning') {
			parallel {
				stage('NPM Dependency Audit') {
					steps {
						sh '''
					   	npm audit --audit-level=critical
					   	echo $?
					 	 '''	
					}		
				}
				stage('OWASP Dependency Check') {
					steps {
						dependencyCheck additionalArguments: '''
			   				--scan \'./\'
			       				--out \'./\'
				   			--format \'ALL\'
			       				--prettyPrint''', odcInstallation: 'OWASP-DepCheck-10'

						dependencyCheckPublisher failedTotalCritical: 1, pattern: 'dependency-check-report.xml'
						junit allowEmptyResults: true, keepProperties: true, stdioRetention: '', testResults: 'dependency-check-junit.xml'
						publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: true, reportDir: './', reportFiles: 'index.html', reportName: 'Dependency CheckHTML Report', reportTitles: 'dependency-check-jenkins.html', useWrapperFileDirectly: true])
					}
				}	
			}
		}
		stage('Unit Testing') {
			steps {

				withCredentials([usernamePassword(credentialsId: 'mongo-db-creds', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]) {

				sh "npm test"
				junit allowEmptyResults: true, keepProperties: true, stdioRetention: '', testResults: 'test-results.xml'
}
				}
		}
	}
}



