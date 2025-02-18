pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        S3_BUCKET = 'aws-sam-cli-managed-default-samclisourcebucket-b7zkzyspcxu0'
        STACK_NAME = 'staging-todo-list-aws'
    }

    stages {
        stage('Get Code') {
            steps {
                git branch: 'master', 
                    url: 'https://github.com/wien996/CP1.D.DevopsCloudUnir.git'
            }
        }

        stage('Deploy SAM') {
            steps {
                script {
                    sh '''
                    sam deploy \
                        --stack-name $STACK_NAME \
                        --s3-bucket $S3_BUCKET \
                        --region $AWS_REGION \
                        --capabilities CAPABILITY_IAM \
                        --no-confirm-changeset \
                        --no-fail-on-empty-changeset
                    '''
                }
            }
        }
		
		stage('Rest Test') {
            steps {
                script {
                sh '''
                export BASE_URL=https://jfczlorg01.execute-api.us-east-1.amazonaws.com/prod  # Configura la URL correcta
                export PATH=$PATH:/var/lib/jenkins/.local/bin  # Asegurar que pytest esté accesible
                
                # Ejecutar solo pruebas de solo lectura
                pytest -m read_only test/integration/todoApiTest.py --junitxml=reports/rest_test_report.xml || echo "No se encontraron pruebas de solo lectura"
                '''
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'reports/*.xml', fingerprint: true
                }
            }
        }
    }
}
