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

    stage("Integration Tests Production") {
      agent {
        docker {
          image 'postman/newman'
          args '--entrypoint='
        }
      }
      when {
        expression { mapBranch[params.DEPLOY_TO] == "production" }
      }
      steps {
        sh 'newman run "https://www.getpostman.com/collections/579ba7bb39193a0682f8" -e "./integration_tests/production.json"'
        echo "Successfully deployed to ${mapBranch[params.DEPLOY_TO]}"
      }
    }

    stage("Integration Tests for QA") {
      agent {
        docker {
          image 'postman/newman'
          args '--entrypoint='
        }
      }
      when {
        expression { mapBranch[params.DEPLOY_TO] == "qa" }
      }
      steps {
        sh 'newman run "https://www.getpostman.com/collections/579ba7bb39193a0682f8" -e "./integration_tests/qa.json"'
        echo "Successfully deployed to ${mapBranch[params.DEPLOY_TO]}"
      }
		}
	}
}