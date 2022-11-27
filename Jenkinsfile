#!/usr/bin/env groovy 

pipeline {

	agent any 
	
	stages {
		stage("increment version"){
			steps{
				script{
					dir("app"){
						echo "Updating the application version..."
					    sh " npm version patch "
					    def packageJson = readJSON file: 'package.json'
					    def version = packageJson.version
					    env.IMAGE_NAME = "solomoncode26/nodejs-app:$version-$BUILD_NUMBER"
					}
				}
			}
		}
		stage("test"){
			steps{
				script{
                    dir("app"){
                    	echo "testing the application..."
                    	sh "npm install"
                    	sh "npm test"
                    }
				}
			}
		}
		stage("build app"){
			steps{
				script{
					dir("app"){
						echo "building the application......."
						sh "npm pack"
					}
				}
			}
		}
		stage("build image"){
			when{
				expression{
				  BRANCH_NAME == 'main'
				}
			}
			steps{
				script{
					echo "building the image"
					withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'PASS', usernameVariable: 'USER')]){
						sh "docker build -t ${IMAGE_NAME} ."
						sh "echo ${PASS} | docker login -u ${USER} --password-stdin"
						sh "docker push ${IMAGE_NAME}"
					}
				}
			}
		}
		stage("deploy"){
			when{
				expression{
				  BRANCH_NAME == 'main'
				}
			}
			steps{
				script{
					echo " deploying docker imagee to EC2..."
					def shellCmd = "bash ./docker.sh ${IMAGE_NAME}"
					sshagent(['ec2-ssh-key']) {
						sh "scp docker.sh ec2-user@18.134.5.157:/home/ec2-user"
						sh "scp docker-compose.yaml ec2-user@18.134.5.157:/home/ec2-user"
						sh "ssh -o StrictHostKeyChecking=no ec2-user@18.134.5.157 ${shellCmd}"
					}   
				}
			}
		}
		stage("commit upate version"){
			steps{
				script{
					withCredentials([usernamePassword(credentialsId: 'gitlab-credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]){
						sh 'git config --global user.email "jenkins@example.com"' 
						sh 'git config --global user.name "jenkins"'

						sh "git remote set-url origin https://${USER}:${PASS}@gitlab.com/solomoncode26/nodejs-complete-ci-cd-pipeline.git"
						sh 'git add .'
						sh 'git commit -m "ci: version bump"'
						sh 'git push origin HEAD:main'
					}
				}
			}
		}
	}
}
