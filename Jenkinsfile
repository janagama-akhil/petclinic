pipeline {
	agent any 
	    stages {
			stage('build') {
				steps {
					sh 'mvn clean package'
				}
			}
			stage('archival') {
				steps {
					archiveArtifacts 'target/*.war'
				}
			}
			stage('junit test cases') {
				steps {
					junit 'target/surefire-reports/*.xml'
				}
			}
			stage('artifact nexus uploader') {
				steps {
					nexusArtifactUploader artifacts: [[artifactId: 'spring-petclinic',
			        classifier: '', file: 'target/petclinic.war', type: 'war']],
					credentialsId: 'nexusid',
					groupId: 'org.springframework.samples',
					nexusUrl: '3.16.163.13:8081/nexus',
					nexusVersion: 'nexus2',
					protocol: 'http',
					repository: 'releases',
					version: "4.2.${BUILD_NUMBER}"
				}
			}

          stage('Deploy') {
          		input {
               		message "Should we continue?"
               		ok "Yes, we should."
            		}
           		steps {
           			checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, 
                    extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'ansible']], submoduleCfg: [], 
        			userRemoteConfigs: [[url: 'https://github.com/janagama-akhil/tomcat-standalone.git']]])
                   
                   withCredentials([string(credentialsId: 'ansi_vault_pass', variable: 'MYPASS')]) {
        			sh '''
                    	                        echo $MYPASS
						echo $MYPASS > ~/.vault_pass.txt
						export ANSIBLE_VAULT_PASSWORD_FILE=~/.vault_pass.txt
						cd ansible
						ansible-playbook -i production -e "BUILD_NO=${BUILD_NUMBER}" --vault-id ~/.vault_pass.txt site.yml 
		              '''
                               }
           		    }
			}
		}

post {
		always {
			notify('started')
		}
		failure {
			notify('err')
		}
		success {
			notify('success')
		}
	}
}


def notify(status) {
			emailext (
				to: 'udu6767@gmail.com',
			    subject: "${status}: JOB:'${env.JOB_NAME} job ${env.JOB_ID}: ${env.JENKINS_HOME}: ${env.BUILD_ID}: --${status}'",
				body: "Please go to ${BUILD_URL} and verify the build"
			)
		}
