pipeline {
    agent any

     stages {
        stage('Get Code') {
            steps {
                // Obtener código del repo
                git 'https://github.com/siati38/helloworld.git'
            }
        }
            
        stage('Unit') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    bat '''
                        set PYTHONPATH=%WORKSPACE%
                        pytest --junitxml=result-unit.xml test\\unit
                    '''
					junit 'result*.xml'
               }
            }
        }   

		stage ('Static') {
			steps {
				bat '''
					flake8 --format=pylint --exit-zero app >flake8.out
				'''
				recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates: [[threshold:8, type: 'TOTAL', unstable: true], [threshold: 10, type: 'TOTAL', unstable: false]]
			}
		}
		
		stage ('Cobertura') {
			steps {
				bat '''
					coverage run --branch --source=app --omit=app\\__init__.py,app\\api.py -m pytest test\\unit
					coverage xml
				'''
				cobertura coberturaReportFile: 'coverage.xml', conditionalCoverageTargets: '80,90,100', lineCoverageTargets: '85,95,100'
                    
			}
		}
		
		stage ('Seguridad') {
			steps {
				bat '''
					bandit --exit-zero -r . -f custom -o bandit.out --severity-level medium --msg-template "{abspath}:{line}: [{test_id}] {msg}"
				'''
				recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], qualityGates: [[threshold:1, type: 'TOTAL', unstable: true], [threshold: 2, type: 'TOTAL', unstable: false]]
			}
		}
		
        stage('Rest') {
            steps {
                bat '''
                    set FLASK_APP=app\\api.py
                    set FLASK_ENV=development
                    start flask run
                    start java -jar C:\\Unir\\Ejercicios\\wiremock\\wiremock-jre8-standalone-2.28.0.jar --port 9090 --verbose --root-dir C:\\Users\\Javi\\Documents\\unir\\helloworld\\test\\wiremock
                    set PYTHONPATH=%WORKSPACE%
                    pytest --junitxml=result-rest.xml test\\rest
                '''
            }    
        }

    }
}
