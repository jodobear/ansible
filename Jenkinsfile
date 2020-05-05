def mapBranch = ["master": "production",
				"qa": "QA"]
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
        script {
					def branch = mapBranch[params.DEPLOY_TO]
					echo "Deploying to ${branch}"
				}
				ansiblePlaybook credentialsId: 'vagrant-toolbox-key', disableHostKeyChecking: true, inventory: "inventories/${params.DEPLOY_TO}/hosts.ini", playbook: 'playbook.yml'
			}
		}
		stage('Integration Tests') {
			agent {
				docker {
					image 'postman/newman'
					args '--entrypoint='
				}
			}
			steps {
				sh 'newman run "https://www.getpostman.com/collections/f4b14bdc4e626adba9ad"'
			}
		}
		stage('Echo Deployment Environment') {
			steps {
				script {
					def branch = mapBranch[params.DEPLOY_TO]
					echo "Successfully deployed to ${branch}"
				}
			}
		}
	}
}
