pipeline {
	agent any
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
				stage('Artifact Uploader') {
					steps {
						
					nexusArtifactUploader artifacts: [[artifactId: 'spring-petclinic', classifier: '', file: 'target/petclinic.war', type: 'war']], credentialsId: 'nexusid', groupId: 'org.springframework.samples', nexusUrl: '3.15.181.60:8081/nexus', nexusVersion: 'nexus2', protocol: 'http', repository: 'releases', version: "4.2.${BUILD_NUMBER}"
					}
				}
			}
		}
		stage('Deploy') {
			input {
                		message "Should we continue?"
                		ok "Yes, we should."
            		}
			steps {
				// git 'https://github.com/akmaharshi/tomcat-standalone.git'
				checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, 
                                          extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'ansible']], submoduleCfg: [], 
        				userRemoteConfigs: [[url: 'https://github.com/janagama-akhil/petclinic.git']]])
				
				
					sh 'sudo ansible-playbook -i production -e "BUILD_NO=${BUILD_NUMBER}" site.yml'
				
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
		to: 'udu6767@gmail.com',
		subject: "JOB:${env.JOB_NAME} with Build: ${env.BUILD_ID} ${status}", 
		body: "${status} - ${env.BUILD_URL}"
	)
}
