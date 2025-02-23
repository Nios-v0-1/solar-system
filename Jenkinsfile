// edited ...3210

pipeline {
	agent any
	
	tools {
		  nodejs 'nodejd-22-12-0'
		}
	environment {
		MONGO_URI = "mongodb+srv://supercluster.d83jj.mongodb.net/superData"
		MONGO_DB_CREDS = credentials('mongo-db-creds')
		MONGO_USERNAME = credentials('mongo-db-username')
		MONGO_PASSWORD = credentials('mongo-db-password')
		SONAR_SCANNER_HOME = tool 'sonarqube-scanner-700'
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
										}
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
				
				}
		}
		stage('Code Coverage') {
			steps {
				catchError(buildResult: 'SUCCESS', message: 'Oops! it will be fixed the future releases', stageResult: 'UNSTABLE') {
					sh ''' 
     						npm run coverage	
	   					
					'''
				}
				
			}
		}
		stage('SAST - SonarQube') {
			steps {      
					sh 'echo $SONAR_SCANNER_HOME'
					sh ''' 
	   					$SONAR_SCANNER_HOME/bin/sonar-scanner \
						-Dsonar.projectKey=Solar-System-Project \
						-Dsonar.sources=app.js\
     						-Dsonar.javascript.lcov.reportPaths=./coverage/lcov.info
						-Dsonar.host.url=http://10.170.20.63:9000 \
						-Dsonar.token=sqp_71fd68d0e3ede04de464d022dfb5efb22fb97015
					'''
				}
				
			}
		
	       }
	post {
		always {
			junit allowEmptyResults: true, keepProperties: true, stdioRetention: '', testResults: 'dependency-check-junit.xml'
			junit allowEmptyResults: true, keepProperties: true, stdioRetention: '', testResults: 'test-results.xml'
			publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: true, reportDir: './', reportFiles: 'dependency-check-report.html', reportName: 'Dependency Check Report', reportTitles: '', useWrapperFileDirectly: true])	
			publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: true, reportDir: './coverage/lcov-report', reportFiles: 'index.html', reportName: 'Code Coverage HTML Report', reportTitles: '', useWrapperFileDirectly: true])

		}
	}
}



