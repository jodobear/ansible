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
		stage('execute playbook in prod') {
      when {
        branch 'master'
      }
			steps {
				ansiblePlaybook credentialsId: 'vagrant-toolbox-key', disableHostKeyChecking: true, inventory: "inventories/prod/hosts.ini", playbook: 'playbook.yml'
			}
		}
    stage('execute playbook in qa') {
      when {
        branch 'qa'
      }
			steps {
				ansiblePlaybook credentialsId: 'vagrant-toolbox-key', disableHostKeyChecking: true, inventory: "inventories/qa/hosts.ini", playbook: 'playbook.yml'
			}
		}
	}
}

