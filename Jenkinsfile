pipeline {
    agent {
        label 'AGENT-1'
        
    }
    options{
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        //retry(1)

    }

    parameters{
        booleanParam(name: 'deploy', defaultValue: false, description: 'Toggle this value')
    }
    environment {
        DEBUG = 'true'
        appVersion = '' // this will become global , we can use across pipeline
        region = 'us-east-1'
        account_id = '557690626059'
        project = 'expense'
        component = 'frontend'
        environment = 'dev'
    }
   
    stages {
        stage('Read the version') { 
            steps {
                script{
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "App version: ${appVersion}"
                }
            }
        }

       
        stage('Docker build') { 
            
            steps {
                withAWS(region: 'us-east-1', credentials: 'aws-creds') {
                    sh """
                    aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${account_id}.dkr.ecr.us-east-1.amazonaws.com

                    docker build -t ${account_id}.dkr.ecr.us-east-1.amazonaws.com/${project}/${environment}/${component}:${appVersion} .

                  

                    docker push ${account_id}.dkr.ecr.us-east-1.amazonaws.com/${project}/${environment}/${component}:${appVersion} 

                    """
                }
            }
        }
        stage('Trigger Deploy'){
            when { 
                expression { params.deploy }
            }
            steps{
                build job: 'frontend-cd', parameters: [string(name: 'version', value: "${appVersion}")], wait: true
            }
        }
    }
      
    

    post {
        always{
            echo "This sections runs always"
            deleteDir()
            
        }
        success{
            echo "This section run when pipeline success"
        }
        failure{
            echo "This section run when pipeline failure"
        }


    }

}
