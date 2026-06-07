pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo "workspace de trabajo ${WORKSPACE}"
                 git branch: 'master', url: 'https://github.com/devrtenesaca/todo-list-aws.git'
            }
        }

        stage('Build') {
            steps {
                script {
                    // Usamos --use-container para evitar problemas de versiones de Python locales
                    sh "ls -la"
                    sh "sam build"
                }
            }
        }

        stage('Deploy') {
            steps {
                // El bloque withAWS gestiona las credenciales automáticamente
                    sh """
                    sam deploy \
                        --config-env production \
                        --resolve-s3 \
                        --no-confirm-changeset \
                        --no-fail-on-empty-changeset
                    """
            }
        }
    
       stage('GetOutputs') {
            steps {
                script {
                    def apiUrl = sh(
                        script: "aws cloudformation describe-stacks --stack-name todo-list-aws-production --query 'Stacks[0].Outputs[?OutputKey==`BaseUrlApi`].OutputValue' --region us-east-1 --output text",
                        returnStdout: true ).trim()
                    // Asignamos a una variable de entorno de Jenkins
                    env.BASE_URL = apiUrl
                    echo "La URL de la API capturada es: ${env.BASE_URL}"
                }
            }
        }  
        
        stage('Unit-Testing') {
            steps {
                echo "ejecutando pruebas unitarias"
                sh '''
                    python3 -m pytest -v test/integration/todoApiTest.py --junitxml=result-api.xml
                '''
                
            }
        }
        
        stage ('Results'){
            steps {
                junit 'result*.xml'
            }
        }
        
  }
}