pipeline {
    agent any
	
    stages {
		stage('Clean up existing data') {
			steps {
				dir('kubernetes-init/data') {
					sh 'rm -rf *'
					//echo "current build number: ${currentBuild.number}"
				}
			}
		}
		stage('Pull Jmeter Image from registery') {
			steps {
			    sh 'docker pull saikrishnaoduru/jmeter'
			}
		}
        stage('Get latest Build from repository') {
            steps {
                echo 'Pulling latest script from GIT'
                git 'https://github.com/Yogendaroduru/jmeter.git'
            }
        }
        stage('Build Kubernetese Config') {
            steps {
                sh 'mkdir -p `pwd`/kubernetes-init/data/;cp `pwd`/data/script.jmx `pwd`/kubernetes-init/data/script.jmx;'
                sh 'cd `pwd`/kubernetes-init/;chmod 775 create.sh;./create.sh 2 script.jmx'
                sh 'mkdir -p `pwd`/kubernetes-init/data/logs/;mkdir -p `pwd`/kubernetes-init/data/results/;'
                echo 'kubernetes configuration for Jmeter Masters and Slaves created successfully'
                //echo "current build number: ${currentBuild.number}"
            }
        }
        stage('Deploying Jmeter Slave Pods') {
            steps {
				sh 'cd `pwd`/kubernetes-init/config/; ls; i=1;while [ "$i" -le 2 ]; do  microk8s kubectl apply -f slave$i.yaml; i=$(( i + 1 )); done'
				sh 'sleep 60;'
                echo 'Started Jmeter Slave Pods'
            }
        }
		stage('Deploying JMeter Master and Start Test') {
            steps {
				echo 'Starting Execution'
                sh 'cd `pwd`/kubernetes-init/config/; microk8s kubectl apply -f master.yaml'
                sh 'sleep 60;'
				sh 'microk8s kubectl logs master -f'
                
            }
        }
        stage('Stopping Jmeter Pods') {
            steps {
                sh 'cd `pwd`/kubernetes-init/config/; ls; microk8s kubectl delete -f .;'
                echo 'Creating Pods for master & slave and Start execution'
            }
        }
		stage('Archive data') {
			steps {
				dir('kubernetes-init/data') {
					archiveArtifacts artifacts: '**'
				}
			}
		}
    }
}
