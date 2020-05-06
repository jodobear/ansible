def mapBranch = ["master": "production",
				          "qa": "qa"]
pipeline {
	agent any

  parameters {
    choice(name: 'DEPLOY_TO',
            choices: ['master', 'qa'],
            description: 'Choose deployment environment')
  }

  environment {
    ENVIRONMENT_NAME = "${mapBranch[params.DEPLOY_TO]}"
  }

	stages {
		stage('copy artifact from go-artifact') {
			steps {
				copyArtifacts filter: 'go-artifact',
        fingerprintArtifacts: true,
        projectName: 'go-artifact',
        selector: lastSuccessful()
			}
		}

		stage('Deliver package & execute playbook') {
			steps {
        echo "Deploying to ${mapBranch[params.DEPLOY_TO]}"
				ansiblePlaybook credentialsId: 'vagrant-toolbox-key',
                disableHostKeyChecking: true,
                inventory: "inventories/${mapBranch[params.DEPLOY_TO]}/hosts.ini",
                playbook: 'playbook.yml'
			}
		}
    stage("Integration Tests") {
      agent {
        docker {
          image 'postman/newman'
          args '--entrypoint='
        }
      }
      steps {
        when {
          expression { mapBranch[params.DEPLOY_TO] == "production"}
        }
        sh 'newman run "https://www.getpostman.com/collections/886f5b6ce9804525359d" -e "./integration_tests/production.json"'
        echo "Successfully deployed to ${mapBranch[params.DEPLOY_TO]}"
      }
		}
	}
}