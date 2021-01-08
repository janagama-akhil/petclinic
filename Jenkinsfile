pipeline {
	agent any
	stages {
		stage('Build') {
			steps {
			sh 'mvn clean package'
    		}
		}
		
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
						
				nexusArtifactUploader artifacts: [[artifactId: 'spring-petclinic', classifier: '',
				file: 'target/petclinic.war', type: 'war']], 
				credentialsId: 'nexusid', groupId: 'org.springframework.samples', 
				nexusUrl: '3.15.181.60:8081/nexus', 
			 	nexusVersion: 'nexus2', 
				protocol: 'http', 
				repository: 'releases', 
				version: "4.2.${BUILD_NUMBER}"
			}
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
				
				        
					sh '''
					    cd tomcat-standalone
					    sudo ansible-playbook -i production -e "BUILD_NO=${BUILD_NUMBER}" site.yml
					   '''
				
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
