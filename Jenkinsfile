stageResultMap = [:]
pipeline {
    agent none 
    environment {
        BUILD_PATH = 'devops-test'
        SETUP = 'python setup.py bdist_wheel'
        INSTALL_REQUIREMENTS = 'pip install -r requirements/development.txt'
        RUN_TESTS = 'pytest tests --cov package'
    }
    stages {
        stage('Build and Tests') {
            agent {label 'master'}
            steps {
               sh '''
               rm -rf ${WORKSPACE}/${BUILD_PATH}
               git clone git@gitlab.com:dw_web_services/devops-test.git
               cd ${WORKSPACE}/${TMP}
               ${SETUP}
               ${INSTALL_REQUIREMENTS}
               ${RUN_TESTS}
               '''
            } 
        }
        stage(Build docker){
            environment { 
                REGISTRY = '127.0.0.1:5000'
                IMAGE_NAME = 'DRWEB_PYTHON'
                TAG = 'latest' }
           agent {label 'master'}
           steps {
               scrip {
                   dockerImage = docker.build('${env.IMAGE_NAME}:${env.TAG')   
               }
           }
        }
        stage('Deploy our image') { 
          steps { 
            script { 
                docker.withRegistry(${REGISTRY}) { 
                    dockerImage.push() 
                    }
                } 
            }
        } 
        stage(Kuber deploy){
            agent {
                kubernetes {
                label 'master'
                yaml """
        kind: Pod
        metadata:
            name: 
        spec:
            containers:
            - name: '${IMAGE_NAME}'
              image: 127.0.0.1:5000/'${IMAGE_NAME}'
"""
   }

        }
   }
}