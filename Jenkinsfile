pipeline {
    agent any

    stages {
        stage('GetCode') {
            steps {
                echo "Workspace directory is: ${WORKSPACE}"
                echo 'Getting code'
                git branch: 'develop', url: 'https://github.com/devrtenesaca/todo-list-aws.git'
            }
        }
        
        stage ('Static Test'){
            parallel {
                
                stage('Static Linting') {
                    steps {
                        sh 'flake8 --exit-zero --format=pylint src > flake8.out'
                            catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                                recordIssues(
                                    tools: [pyLint(name: 'Flake8-StaticReport', pattern: 'flake8.out')],
                                    qualityGates: [
                                        [threshold: 8, type: 'TOTAL', unstable: true],
                                        [threshold: 10, type: 'TOTAL', unstable: false]
                                    ]
                                )
                        }
                    }
                }
                
                stage('Security Test') {
                    steps {
                        sh '''
                            bandit --exit-zero -r . -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id}] {msg}"
                        '''
                        recordIssues(
                            tools: [pyLint(id: 'bandit', name: 'Bandit Security', pattern: 'bandit.out')]
                        )
                    }
                }
                
            }
        }
        
        
        stage('Build') {
            steps {
                script {
                    echo "stage:  build infraestructura"
                    sh "ls -la"
                    sh "sam build"
                }
            }
        }

        stage('Deploy') {
            steps {
                echo "stage: deployando infraestructura"
                    sh """
                    sam deploy \
                        --config-env staging \
                        --resolve-s3 \
                        --no-confirm-changeset \
                        --no-fail-on-empty-changeset
                    """
            }
        }
    
       stage('GetOutputs') {
            steps {
                echo "stage: obteniendo outputs"
                script {
                    def apiUrl = sh(
                        script: "aws cloudformation describe-stacks --stack-name todo-list-aws-staging --query 'Stacks[0].Outputs[?OutputKey==`BaseUrlApi`].OutputValue' --region us-east-1 --output text",
                        returnStdout: true ).trim()
                    // Asignamos a una variable de entorno de Jenkins
                    env.BASE_URL = apiUrl
                    echo "La URL de la API capturada es: ${env.BASE_URL}"
                }
            }
        }  
        
        stage('Unit-Testing Deploy') {
            steps {
                echo "stage: ejecutando pruebas unitarias"
                sh '''
                    export BASE_URL=${BASE_URL}
                    python3 -m pytest -v test/integration/todoApiTest.py --junitxml=result-api.xml
                '''
                
            }
        }
        
        stage ('Results'){
            steps {
                echo "stage: visualizando resultados"
                junit 'result*.xml'
            }
        }
        
        stage('Auto Merge') {
            when {
                expression { currentBuild.currentResult == 'SUCCESS' }
            }
        
            steps {
                sh '''
                    git config user.email "rteneasca@opensip.tech"
                    git config user.name "devrtenesaca"
                    git config merge.ours.driver true
                    
                    git checkout master
                    git pull origin master
                    
                    git merge origin/develop --no-commit --no-ff 
                    git checkout HEAD -- Jenkinsfile
                    git commit -m "Auto merge develop -> master (Pipeline protegido)" || echo "No hay cambios para commitear"
                    git push origin master
                '''
            }
        }
    
    }
}
