pipeline {
    agent any
    parameters {
        string(name: 'ENVIRONMENT', defaultValue: 'dev', description: 'Specify the environment for deployment')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: "Run Tests in pipeline")
    
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
                sh 'source venv/bin/activate'
                 sh 'pip3 install -r requirements.txt'
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
                sh "pytest"
                
            }
        }         
            
    }
}
