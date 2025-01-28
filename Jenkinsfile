// edited ...3210
pipeline {
	agent any
	
	tools {
		  nodejs 'nodejd-22-12-0'
		}
	environment {
		MONGO_URI = "mongodb+srv://supercluster.d83jj.mongodb.net/superData"
		MONGO_DB_CREDS = credentials('mongo-db-creds')
	}
	options {
		disableResume()
		disableConcurrentBuilds abortPrevious:true
	}

	stages {
		stage('Installing Dependencies') {
			options {
				  timestamps ()
			}
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
						publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: true, reportDir: './', reportFiles: 'dependency-check-report.html', reportName: 'Dependency Check Report', reportTitles: '', useWrapperFileDirectly: true])					}
				}	
			}
		}
		stage('Unit Testing') {
			options { 
				retry(3)
				}
			steps {
				sh 'echo colon separated $MONGO_DB_CREDS'
				sh 'echo username - $MONGO_DB_CREDS_USR'
				sh 'echo Password - $MONGO_DB_CREDS_PSW'
				sh "npm test || true "
				junit allowEmptyResults: true, keepProperties: true, stdioRetention: '', testResults: 'test-results.xml'
				
				}
		}
		stage('Code Coverage') {
			steps {
				catchError(buildResult: 'SUCCESS', message: 'Oops! it will be fixed the future releases', stageResult: 'UNSTABLE') {
					sh "npm run coverage"
				}
				
				publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: true, reportDir: './coverage/lcov-report', reportFiles: 'index.html', reportName: 'Code Coverage HTML Report', reportTitles: '', useWrapperFileDirectly: true])
				}
		}
	}
}



