def GIT_URL = "git@github.com:TapAleksej/nuxt.git"

pipeline {
	// глобальный агент
	agent {
		label 'docker'
	}
	environment {
		PRJ_DIR = "/var/www/nuxt"
		// jenkins-stage
		HOST = "89.169.184.123"
		BRANCH = "main"
	}
	stages {
		stage('Checkout') {
			steps {
				git branch: "${env.BRANCH}", url: "${GIT_URL}"
			}			 
		}
		stage('Build static') {
			// Переопределяем агента вместо глобального
			agent {
				docker {
					image 'node'
					label 'docker'
					reuseNode: true
				}
			}
			steps {
				// запуск проги 1:13
				sh "npm ci --cache .npm --prefer-offline"
				sh "npm run generate"
			}
		}
		stage('Deliviry') {
			steps {
				script {
					sh """
						set -ex;
						rsync -av --delete -e "ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -i '${private_key}'" .output/public "${username}@${env.HOST}:${env.PRJ_DIR}/"
						"""
					sh "tar -czvf frontend.tar.gz .output/public"
					archiveArtifacts artifacts: 'frontend.tar.gz', fingerprint: true, onlyIfSuccessful: true	
				}
			}
		}		
	}
}
