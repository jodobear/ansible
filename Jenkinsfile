def output
pipeline {
	agent any

	stages {
		stage('copy artifact from go-artifact') {
			steps {
				copyArtifacts filter: 'go-artifact', fingerprintArtifacts: true, projectName: 'go-artifact', selector: lastSuccessful()
			}
		}
		stage('execute playbook') {
			steps {
				ansiblePlaybook credentialsId: 'vagrant-toolbox-key', disableHostKeyChecking: true, inventory: 'hosts.ini', playbook: 'playbook.yml'
			}
		}
	}
}

