def mapBranch = ["master": "production",
				          "qa": "qa"]
pipeline {
	agent any

  parameters {
    choice(name: 'DEPLOY_TO',
            choices: ['master', 'qa'],
            description: 'Choose deployment environment')
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
        script {
          def test_env = ${mapBranch[params.DEPLOY_TO]}
        }
        echo "${test_env}"
        // sh 'newman run "https://www.getpostman.com/collections/886f5b6ce9804525359d" -e "./integration_tests/$test_env.json"'
        echo "Successfully deployed to ${mapBranch[params.DEPLOY_TO]}"
      }
		}
	}
}