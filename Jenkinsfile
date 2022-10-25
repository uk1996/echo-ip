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
    
  agent {
    label 'docker' 
  }
  stages {
      steps {
        git url: 'https://github.com/uk1996/echo-ip.git', branch: 'main'
      }
    }
    stage('docker build and push') {
      agent {
        docker {
          // Set both label and image
          label 'docker'
          image 'node:7-alpine'
          args '--name docker-node' // list any args
        }
      }
      steps {
        // Steps run in node:7-alpine docker container on docker agent
        sh 'node --version'
      }
    }

    stage('Docker maven test') {
      agent {
        docker {
          // Set both label and image
          label 'docker'
          image 'maven:3-alpine'
        }
      }
      steps {
        // Steps run in maven:3-alpine docker container on docker agent
        sh 'mvn --version'
      }
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
