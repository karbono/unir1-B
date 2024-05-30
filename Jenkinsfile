pipeline {
    agent any
    stages {
        stage('Get Code') {
            steps {
                git 'https://github.com/karbono/unir1-B'
            }
        }
        stage('Cobertura') {
            steps {
                bat '''
                    SET PYTHONPATH=%WORKSPACE%
                    coverage run --branch --source=app --omit=app\\__init__.py,app\\api.py -m pytest --junitxml=result-unit.xml test\\unit
                    coverage xml
                '''
                junit 'result-unit.xml'
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    cobertura coberturaReportFile: 'coverage.xml', onlyStable: false, failUnstable: false, 
                        conditionalCoverageTargets: '100,80,90', lineCoverageTargets: '100,85,95'
                }
            }
        }
        stage('Rest') {
            steps {
                bat '''
                    set FLASK_APP=app\\api.py
                    start /B flask run
                    start /wait timeout 5
                    start /B java -jar C:\\Users\\lacei\\Downloads\\wiremock-standalone-3.5.3.jar --port 9090 --root-dir test\\wiremock
                    start /wait timeout 5
                    SET PYTHONPATH=%WORKSPACE%
                    pytest --junitxml=result-rest.xml test/rest
                '''
                }
            }
        stage('Static') {
            steps {
                bat '''
                    flake8 --exit-zero --format=pylint app > flake8.out
                '''
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    recordIssues(
                        tools: [flake8(name: 'Flake8', pattern: 'flake8.out')],
                        qualityGates: [
                            [threshold: 8, type: 'TOTAL', unstable: true],
                            [threshold: 10, type: 'TOTAL', unhealthy: true]
                        ]
                    )
                }
            }
        }
        stage('Security') {
            steps {
                bat '''
                    bandit --exit-zero -r . -f custom -o bandit.out --severity-level medium --msg-template "{abspath}:{line}: [{test_id}] {msg}"
                '''
                catchError(buildResult: 'FAILURE', stageResult: 'FAILURE') {
                    recordIssues(
                        tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')],
                        qualityGates: [
                            [threshold: 2, type: 'TOTAL', unstable: true],
                            [threshold: 4, type: 'TOTAL', unhealthy: true]
                        ]
                    )
                }
            }
        }
        stage('Performance') {
            steps {
                bat 'C:\\Users\\lacei\\Downloads\\apache-jmeter-5.6.3\\bin\\jmeter -n -t test\\jmeter\\flask.jmx -f -l flask.jtl'
                perfReport sourceDataFiles: 'flask.jtl'
            }
        }
    }
}