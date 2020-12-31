pipeline {
	agent { label 'myagent' }
	environment {
		def tomcatDevIp = '18.188.216.38'
		def tomcatHome = '/home/ec2-user/apache-tomcat-8.5.61'
        	def tomcatStart = "${tomcatHome}/bin/startup.sh"
        	def tomcatStop = "${tomcatHome}/bin/shutdown.sh"
	}
	stages {
    	// stage('My Parallel stages') {
    	//	parallel {
    			/* stage('SonarQube analysis') { 
    				steps {
						withSonarQubeEnv('Sonar') { 
						sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.3.0.603:sonar ' + 
						'-Dsonar.projectKey=com.petclinic:all:master ' +
						'-Dsonar.language=java ' +
						'-Dsonar.sources=. ' +
						'-Dsonar.tests=. '
						}
					}
				} */
				stage('Build') {
					steps {
    					sh 'mvn clean package'
    				}
				}
			//}
		//}

		/* stage('Check Quality gates') {
			steps {
				script {
					timeout(time: 1, unit: 'HOURS') {
					sleep 30
					def qg = waitForQualityGate() 
						if (qg.status != 'OK') {
							error "Pipeline aborted due to quality gate failure: ${qg.status}"
						}
					}
				}
			}
		} */
		stage('Post Build Actions') {
    		parallel {
				stage('Archive') {
					steps {
						archiveArtifacts artifacts: 'target/*.?ar', followSymlinks: false
					}	
				}
				stage('Unit tests') {
					steps {
						junit 'target/surefire-reports/*.xml'
					}
				}
			}
		}
		stage('Deploy') {
			steps {
				script {
					//sshagent (credentials: ['tomcat']) {
					//	sh "scp -o StrictHostKeyChecking=no target/petclinic.war ec2-user@${tomcatDevIp}:${tomcatHome}/webapps/petclinic.war"
                			//	sh "ssh sonar@${tomcatDevIp} ${tomcatStop}"
                			//	sh "ssh sonar@${tomcatDevIp} ${tomcatStart}"
					
				sh '''
				cp -r target/petclinic.war ${tomcatHome}/webapps/petclinic.war
				${tomcatStop}
				${tomcatStart}
				'''
            				//}
            			}
			}
		}
	}
	post {
		success {
			notify('Success')
		}
		failure {
			notify('Failed')
		}
		aborted {
			notify('Aborted')
		}
	}
}

def notify(status) {
	emailext (
		to: 'udu6767@gmail.conm',
		subject: "JOB:${env.JOB_NAME} with Build: ${env.BUILD_ID} ${status}", 
		body: "${status} - ${env.BUILD_URL}"
	)
}
