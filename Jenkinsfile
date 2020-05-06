def mapBranch = ["master": "production",
				"qa": "QA"]
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
        script {
          env.ENV = mapBranch[params.DEPLOY_TO]
          echo "Deploying to ${env.EMV}"
        }
				ansiblePlaybook credentialsId: 'vagrant-toolbox-key',
                disableHostKeyChecking: true,
                inventory: "inventories/${params.DEPLOY_TO}/hosts.ini",
                playbook: 'playbook.yml'
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
        if (params.DEPLOY_TO = "master") {
          sh 'newman run "https://www.gketpostman.com/collections/886f5b6ce9804525359d" -e "./integration_tests/jenkins-deploy-IT-production_env.json"'
        }
        if (params.DEPLOY_TO = "qa") {
          sh 'newman run "https://www.gketpostman.com/collections/886f5b6ce9804525359d" -e "./integration_tests/jenkins-deploy-IT-qa_env.json"'
        }
        echo "Successfully deployed to ${env.ENV}"
      }
		}
	}
}
