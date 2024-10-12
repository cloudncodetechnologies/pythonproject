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
                echo "testing application"
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
            
  }
}
