def mapBranch = ["master": "production",
				          "qa": "QA"]
pipeline {
	agent any

  parameters {
    choice(name: 'DEPLOY_TO',
            choices: ['master', 'qa'],
            description: 'Choose deployment environment')
  }

  environment {
    ENV = defineEnv()
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
      script {
        def defineEnv() {
          def branch = ${params.DEPLOY_TO}
          if (branch == "master") {
            return 'production'
          }
          else {
            return 'qa'
          }
        }
      }
			steps {
        // script {
          // env.ENV = mapBranch[params.DEPLOY_TO]
        echo "Deploying to ${env.EMV}"
        // }
				ansiblePlaybook credentialsId: 'vagrant-toolbox-key',
                disableHostKeyChecking: true,
                inventory: "inventories/${env.ENV}/hosts.ini",
                playbook: 'playbook.yml'
			}
		}
    stage("Integration Tests in ${env.ENV}") {
      agent {
        docker {
          image 'postman/newman'
          args '--entrypoint='
        }
      }
      when {
        expression { env.ENV == "production" }
      }
      steps {
        sh 'newman run "https://www.gketpostman.com/collections/886f5b6ce9804525359d" -e "./integration_tests/jenkins-deploy-IT-production_env.json"'
        echo "Successfully deployed to ${env.ENV}"
      }
      when {
        expression { env.ENV == "qa" }
      }
      steps {
        sh 'newman run "https://www.gketpostman.com/collections/886f5b6ce9804525359d" -e "./integration_tests/jenkins-deploy-IT-qa_env.json"'
        echo "Successfully deployed to ${env.ENV}"
      }
		}
	}
}
