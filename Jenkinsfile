def output
pipeline {
	agent any
	
	stages {
		stage('execute copy from ansible') {
			steps{
				ansiblePlaybook credentialsId: 'vagrant-toolbox-key', disableHostKeyChecking: true, inventory: 'hosts.ini', playbook: 'playbook.yml'			
			}
		}
	}
}

