pipeline {
    agent any
    parameters {
        string(name: 'ENVIRONMENT', defaultValue: 'dev', description: 'Specify the environment for deployment')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: "Run Tests in pipeline")
    }

    environment {
	    IMAGE_NAME = 'cloudncodetechnologies/pythonproject'
	    IMAGE_TAG = "${IMAGE_NAME}:${env.GIT_COMMIT}"
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: '1b256f95-2b17-4dfa-b6a6-9be1b3d86afd', url: 'https://github.com/cloudncodetechnologies/pythonproject.git'
            }
        }
        
        stage('Setup') {
            steps {
                sh 'python3 -m venv venv'
                sh 'bash -c "source venv/bin/activate && pip install -r requirements.txt"'
            }
        }
        
        stage('Unit test') {
            when {
                expression {
                    params.RUN_TESTS == true
                }
            }
            steps {
                echo "Testing application"
                sh 'bash -c "source venv/bin/activate && pytest"'
            }
        }

        stage('Login to Docker hub') {
	        steps {
		        withCredentials([usernamePassword(credentialsId: 'docker-creds',
		        usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
		        sh 'echo ${PASSWORD} | docker login -u ${USERNAME} --password-stdin' 
		        echo "Login Successful"
		    }
	    }
   	}

        stage('Building Docker image') {
		    steps {
			    sh 'docker build -t ${IMAGE_TAG} .'
			    echo "Docker image build successful"
			    sh 'docker image ls'
		    }
	    }

        stage('Pushing Docker image') {
		    steps {
			    sh 'docker push ${IMAGE_TAG}'
                echo "Docker image pushed successfully"
		    }
	    }

        stage('Deploy to Production') {
            when {
                expression {
                    params.ENVIRONMENT == 'prod'
                }
            }
            steps {
                script {
                    echo "Deploying to Production"
                    withCredentials([
                        sshUserPrivateKey(credentialsId: 'prod-server-ssh', keyFileVariable: 'SSH_KEY'),
                        string(credentialsId: 'prod-server-ip', variable: 'SERVER_IP')
                    ]) {
                        sh '''
                            ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ubuntu@${SERVER_IP} << EOF
                            docker login -u ${USERNAME} -p ${PASSWORD}
                            docker pull ${IMAGE_TAG}
                            docker stop pythonproject || true
                            docker rm pythonproject || true
                            docker run -d --name pythonproject -p 80:80 ${IMAGE_TAG}
                            docker image prune -f
                            EOF
                        '''
                        echo "Deployment to production successful"
                    }
                }
            }
        }
    }
}
