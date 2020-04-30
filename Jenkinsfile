def output
pipeline {
	agent any
	
	stages {
		stage('execute copy from ansible') {
			ansiblePlaybook credentialsId: 'vagrant-toolbox-key', inventory: 'hosts.ini', playbook: 'playbook.yml'			
		}
	}
}

