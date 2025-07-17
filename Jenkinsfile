pipeline {
    agent {
        label 'agent-1-label'
    }

    environment {
        poroject = 'expense'
        env_name = 'dev'
        component = 'frontend'
        app_version = ''
        acc_id = '343430925817'
        region = 'us-east-1'
    }

    parameters {
        booleanParam(name: 'BuildImage', defaultValue: false, description: 'Please check the box to build the image')

        booleanParam(name: 'deploy', defaultValue: false, description: 'Select to deploy or not')

    }

    options {
        timeout(time: 40, unit: 'MINUTES')
        ansiColor('xtrem')
        disableConcurrentBuilds()        
    }

    stages {

        stage('Read app version') {
            steps {
                script {
                    def packageJson = readJSON file: 'scripts/package.json'
                    app_version = packageJson.version
                    echo "app Version: ${app_version}"
                }

            }
        }

        stage('Build image') {
            when {
                expression {
                    params.BuildImage
                }
            }
            steps {
                withAWS(region: 'us-east-1', credentials: 'aws-auth') {
                    sh """
                        aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 343430925817.dkr.ecr.us-east-1.amazonaws.com
                        
                        docker build -t ${acc_id}.dkr.ecr.${region}.amazonaws.com/expense-dev/frontend:${app_version} .

                        docker images

                        docker push ${acc_id}.dkr.ecr.${region}.amazonaws.com/expense-dev/frontend:${app_version}
                    """
                }
            }
        }

        stage('Trigger the Deploy') {
            when {
                expression {
                    params.deploy
                }
            }
            steps {
                build job: "frontend-cd", parameters: [string(name: "image_version", value: "${app_version}"), string(name: "env_name", value: "dev")], wait: false
            }
        }

    }

    post {
        always{
            echo 'this will run always'
            deleteDir()
        }
        success{
            echo 'this will run on success'
        }
        failure{
            echo 'this will run at failure'
        }
    }

}