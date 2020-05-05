def output
pipeline {
	agent any

  parameters {
    choice(name: 'DEPLOY_TO', choices: ['master', 'qa'], description: 'Choose deployment environment')
  }
	stages {
		stage('copy artifact from go-artifact') {
			steps {
				copyArtifacts filter: 'go-artifact', fingerprintArtifacts: true, projectName: 'go-artifact', selector: lastSuccessful()
			}
		}
		stage('Deliver package & execute playbook') {
			steps {
        echo "Deploying to env: ${params.DEPLOY_TO}"
				ansiblePlaybook credentialsId: 'vagrant-toolbox-key', disableHostKeyChecking: true, inventory: "inventories/${params.DEPLOY_TO}/hosts.ini", playbook: 'playbook.yml'
			}
		}
	}
}
