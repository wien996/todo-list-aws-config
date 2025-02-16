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
                git branch: 'develop', 
                    url: 'https://github.com/wien996/CP1.D.DevopsCloudUnir.git'
            }
        }

        stage('Static Test') {
            steps {
                script {
                    sh 'mkdir -p reports'
                    sh 'python3 -m flake8 src --output-file=reports/flake8_report.txt || true'
                    sh 'python3 -m bandit -r src -o reports/bandit_report.json -f json || true'
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'reports/*.txt, reports/*.json', fingerprint: true
                }
            }
        }

        stage('Build SAM') {
            steps {
                script {
                    sh 'sam build'
                }
            }
        }

        stage('Validate SAM') {
            steps {
                script {
                    sh 'sam validate --region $AWS_REGION'
                }
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
                    export PATH=$PATH:/var/lib/jenkins/.local/bin  # Añadir pytest al PATH
                    pytest test/integration/todoApiTest.py --junitxml=reports/rest_test_report.xml
                    '''
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'reports/*.xml', fingerprint: true
                }
            }
        }
        
        stage('Promote') {
			steps {
				script {
					sh '''
        				# Obtener los últimos cambios
                        git checkout master
                        git pull origin master
        
                        # Merge con develop y resolver conflictos automáticamente en caso de archivos de texto simples
                        git merge --no-ff develop -m "Promotion to production" || git merge --abort
        
                        # Si el merge fue exitoso, subir cambios
                        git push origin master || echo "No hay cambios para subir"
					'''
				}
			}
		}

    }
}
