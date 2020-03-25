pipeline {
    agent none
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'python:2-alpine'
                }
            }
            steps {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
                stash(name: 'compiled-results', includes: 'sources/*.py*')
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'qnib/pytest'
                }
            }
            steps {
                sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
        stage('Deliver') { 
            agent { 
               docker { 
		image 'cdrx/pyinstaller-linux:python2'
               }
            }
               environment { 
                  VOLUME = '$(pwd)/sources:/src'
                  IMAGE = 'cdrx/pyinstaller-linux:python2'
              }
            steps {
                dir(path: env.BUILD_ID) { 
                    unstash(name: 'compiled-results') 
                    sh 'pyinstaller -F sources/add2vals.py'
                }
            }
            post {
                success {
                    #archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals" 
                    archiveArtifacts 'dist/add2vals'
                }
            }
        }
    }
}
