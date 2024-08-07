pipeline{
	agent any 
	tools{
		jdk 'jdk-17'
		nodejs 'node16'
	}
	environment{
		SCANNER_HOME=tool 'sonar-token'
	}
	stages{
		stage('clean workspace'){
			steps{
				cleanWs()
			}
		}
		stage('Code Checkout From Git'){
			steps{
				git branch: 'main', url: 'https://github.com/DowlaTitan/AWS-Terraform-Zomato-Project.git'
			}
		}
		stage("SonarQube Code Analysis"){
			steps{
				withSonarQubeEnv('sonar-token'){
					sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=zomato \
                    -Dsonar.projectKey=zomato '''
				}
			}
		}
		stage("Code Quality Gates"){
			steps{
				script{
					 timeout(time: 2, unit: 'MINUTES'){
					waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
				}
			}
		}
	}
		stage("Install Dependencies"){
			steps{
				sh "npm install"
			}
		}
		stage("OWASP FS SCAN"){
			steps{
				dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
				dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
			}
		}
		stage("TRIVY FS SCAN"){
			steps{
				sh "trivy fs . > trivy.txt"
			}
		}
		stage("DOcker Image Build and Push"){
			steps{
				script{
				withDockerRegistry(credentialsId: 'Docker', toolName: 'docker'){
					sh "docker build -t cloudzomato1 . "
					sh "docker tag cloudzomato dowla786/cloudzomato1:latest"
					sh "docker push dowla786/cloudzomato1:latest"
						}
					}
				}
			}
		stage("TRIVY is Image Scanning"){
			steps{
				sh "trivy image dowla786/cloudzomato1:latest >trivy.txt"
			}
		}
		stage("Creating Docker Container "){
			steps{
				sh 'docker run -d --name zomato-app -h zomato -p 3000:3000 dowla786/cloudzomato1:latest'
			}
		}
		stage ("email notification"){
			steps{
				mail bcc: '', body: '''Hello team,jenkins job is success.''', cc: '', from: '', replyTo: '', subject: 'jenkins job', to: 'dowlatitan@gmail.com'
			}
		}
	}
}
