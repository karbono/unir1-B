pipeline {
    agent any

    stages {
        stage('Get Code') {
            steps {
                git 'https://github.com/karbono/unir1-a.git'
            }
        }
        stage('Build') {
            steps {
                echo 'Eyy esto es Python, nada que compilar!'
                echo WORKSPACE
                bat 'dir'
            }
        }
        stage('Tests') {
            parallel {
                
                stage('Unit') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            bat '''
                                set PYTHONPATH=%WORKSPACE%
                                pytest --junitxml=result-unit.xml test/unit
                            '''
                        }
                    }
                }
                stage('Rest') {
                    steps {
                        bat '''
                            set FLASK_APP=app\\api.py
                            start flask run
                            start java -jar C:\\Users\\lacei\\Downloads\\wiremock-standalone-3.5.3.jar --port 9090 --root-dir test\\wiremock
                        
                            SET PYTHONPATH=%WORKSPACE%
                            pytest --junitxml=result-rest.xml test/rest
                            '''
                    }
                }
            }
        }
        stage('Results') {
            steps {
                junit 'result*.xml'
                echo 'SUCCESS!!'
            }
        }
    }
}
