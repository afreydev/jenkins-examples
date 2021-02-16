# jenkins-examples
```
@Library('pipeline-library-demo') _

pipeline {
  agent any
  environment {
    MAX_LINE_LENGTH = "110"
  }
  stages {
    
    stage('cleaning') {
      steps {
        echo "Cleaning"
        echo "workspace: ${env.WORKSPACE}"
        cleanWs()
        sh 'ls -l'
      }
    }
      
    stage('checkout') {
        steps {
            git 'https://github.com/afreydev/python-printer.git'
            sh 'ls -l'
            sh 'pwd'
        }
    }
    
    stage('linter') {
        agent {
            docker {
                image 'python:3.7.2'
                args '-u root'
                reuseNode true
            }
        }
        steps {
            sh 'pip install flake8'
            sh 'flake8 . --count --select=E9,F63,F7,F82 --exclude=setup.py --show-source --statistics'
            sh "flake8 . --count --exclude=setup.py --max-line-length=${MAX_LINE_LENGTH} --statistics"
        }
    }
    stage('build') {
        agent {
            docker {
                image 'python:3.7.2'
                args '-u root'
                reuseNode true
            }
        }
        steps {
            sh 'pip list > list.txt'
            sh 'pip install setuptools pip wheel'
            sh 'python3 setup.py sdist bdist_wheel'
        }
    }
    stage('test') {
        agent {
            docker {
                image 'python:3.7.2'
                args '-u root'
                reuseNode true
            }
        }
        steps {
            sh 'pip install pytest'
            sh 'python setup.py install'
            sh 'pytest'
            sayHello 'Angel'
        }
    }

  }
}
```
```
pipeline {
  agent any
  environment {
    registry = "afreydev/flask-backend"
    registryCredential = 'docker-hub'
    dockerImage = ''
  }
  stages {
    
    stage('cleaning') {
      steps {
        echo "Cleaning"
        echo "workspace: ${env.WORKSPACE}"
        cleanWs()
        sh 'ls -l'
      }
    }
      
    stage('checkout') {
        steps {
            git changelog: false, credentialsId: 'jenkins-local', poll: false, url: 'https://github.com/afreydev/flask-backend.git'
            sh 'ls -l'
            sh 'pwd'
        }
    }
    
    stage('Building image') {
      steps{
        script {
          dockerImage = docker.build registry + ":$BUILD_NUMBER"
        }
      }
    }
    
    stage('Publishing image') {
        steps {
            script {
                docker.withRegistry( '', registryCredential ) {
                    dockerImage.push()
                }
            }
        }
    }

  }
}
```
