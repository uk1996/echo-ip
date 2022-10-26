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
    kubernetes {
      label 'sample-app'
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
metadata:
labels:
  component: ci
spec:
  # Use service account that can deploy to all namespaces
  serviceAccountName: cd-jenkins
  containers:
  - name: golang
    image: golang:1.10
    command:
    - cat
    tty: true
  - name: gcloud
    image: gcr.io/cloud-builders/gcloud
    command:
    - cat
    tty: true
  - name: kubectl
    image: gcr.io/cloud-builders/kubectl
    command:
    - cat
    tty: true
"""
}
  }
    
  stages {
    stage('git scm update') {
      
      steps {
        git url: 'https://github.com/uk1996/echo-ip.git', branch: 'main'
      }
    }
    
    
    stage('docker build and push') {
      steps {
        container('gcloud'){
              sh "PYTHONUNBUFFERED=1 gcloud builds submit -t ."
        }
      }
    }
//     stage('deploy kubernetes') {
//       steps {
//         container('kustomize') {
//           sh '''
//           kubectl create deployment pl-bulk-prod --image=sysnet4admin/echo-hname
//           kubectl expose deployment pl-bulk-prod --type=LoadBalancer --port=8080 \
//                                                  --target-port=80 --name=pl-bulk-prod-svc
//           '''
//         }
//       }
//     }
  }
}
