pipeline {
  environment {
    PROJECT = "phonic-realm-360311"
    APP_NAME = "gceme"
    FE_SVC_NAME = "${APP_NAME}-frontend"
    CLUSTER = "jenkins-cd"
    CLUSTER_ZONE = "asia-northeast3-a"
    IMAGE_TAG = "gcr.io/${PROJECT}/${APP_NAME}:${env.BRANCH_NAME}.${env.BUILD_NUMBER}"
    JENKINS_CRED = "${PROJECT}"
  }
  
  agent any
  stages {
    stage('git scm update') {
      
      steps {
        git url: 'https://github.com/uk1996/echo-ip.git', branch: 'main'
      }
    }
    stage('docker build and push') {
      steps {
        sh '''
        docker build -t cswook96/echo-ip .
        docker push cswook96/echo-ip
        '''
      }
    }
    stage('deploy kubernetes') {
      steps {
        sh '''
        kubectl create deployment pl-bulk-prod --image=cswook96/echo-ip
        kubectl expose deployment pl-bulk-prod --type=LoadBalancer --port=8080 \
                                               --target-port=80 --name=pl-bulk-prod-svc
        '''
      }
    }
  }
}
