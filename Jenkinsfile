pipeline {
    agent none
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
                sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
        stage('Deploy') {
            agent any
            environment { 
                VOLUME = '$(pwd)/sources:/src'
                IMAGE = 'cdrx/pyinstaller-linux:python2'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'SSH_TO_SERVER', usernameVariable: 'SSH_USERNAME', passwordVariable: 'SSH_PASSWORD')]) {
                    script{
                        // Menunggu 1 menit
                        sleep time: 1, unit: 'MINUTES'


                        def remote = [:]
                        remote.name = 'Server'
                        remote.host = '172.16.138.59'
                        remote.user = SSH_USERNAME
                        remote.password = SSH_PASSWORD
                        remote.allowAnyHosts = true

                        // Create directory
                        sshCommand remote: remote, command: """
                              mkdir -p /data/simple-python
                        """ 

                        // Put files
                        sshPut remote: remote, from: 'sources', into: '/data/simple-python'
                        sshPut remote: remote, from: 'sources', into: '/data/simple-python'


                    }
                }
            }
        }

    }
}