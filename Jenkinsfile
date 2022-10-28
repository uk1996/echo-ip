pipeline {
  environment {
    PROJECT = "phonic-realm-360311"
    BUILD_NUM = "${env.BUILD_NUMBER}"
    BRANCH_NAME = "${env.BRANCH_NAME}"
    IMAGE_NAME = "asia-northeast3-docker.pkg.dev/phonic-realm-360311/test-img-registry/quickstart-image:${BRANCH_NAME}_${BUILD_NUM}"
    JENKINS_CRED = "${PROJECT}"
  }
  
  agent {
    kubernetes {
      yaml '''
      apiVersion: v1
      kind: Pod
      metadata:
        labels:
          app: jenkins-agent
        name: jenkins-agent
      spec:
        containers:
        - name: kubectl
          image: gcr.io/cloud-builders/kubectl
          tty: true
          command:
          - cat
        - name: docker
          image: docker:dind
          tty: true
          volumeMounts:
          - mountPath: /var/run/docker.sock
            name: docker-sock
          command:
          - cat
        - name: gcloud
          image: gcr.io/cloud-builders/gcloud
          tty: true
          command:
          - cat
        serviceAccount: cd-jenkins
        volumes:
        - name: docker-sock
          hostPath:
            path: /var/run/docker.sock
      '''
    }
  }
  
    
  stages {
    stage('git scm update') {
      steps {
        git url: 'https://github.com/uk1996/echo-ip.git', branch: 'main'
      }
    }
    stage('define cluster'){
      steps {
        script {
          if(env.BRANCH_NAME == "dev"){
            env.CLUSTER = "tilda-saas-dev"
            env.CLUSTER_ZONE = "asia-northeast3"
          } else if(env.BRANCH_NAME == "prod"){
            env.CLUSTER = "tilda-saas-prod"
            env.CLUSTER_ZONE = "asia-northeast3"
          } else {
            env.CLUSTER = "jenkins-cd"
            env.CLUSTER_ZONE = "asia-northeast3-a"
          }
        }
      }
    }
    stage('set auth') {
      steps {
        container('gcloud') {
          script {
            env.GCLOUD_AUTH=sh(returnStdout: true, script: 'gcloud auth print-access-token').trim()
          }
        }
      }
    }
    stage('docker build and push') {
      steps {
         container('docker'){
              sh '''
              docker login -u oauth2accesstoken -p ${GCLOUD_AUTH} https://asia-northeast3-docker.pkg.dev
              docker build -t ${IMAGE_NAME} .
              docker push ${IMAGE_NAME}
              '''
        }
      }
    }
    stage('deploy kubernetes') {
      steps {
        container('kubectl') {
          sh "sed -i.bak 's#set_image#${IMAGE_NAME}#' ./deployment.yaml"
          step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT, clusterName: env.CLUSTER, zone: env.CLUSTER_ZONE, manifestPattern: './deployment.yaml', credentialsId: env.JENKINS_CRED, verifyDeployments: false])
          step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT, clusterName: env.CLUSTER, zone: env.CLUSTER_ZONE, manifestPattern: './service.yaml', credentialsId: env.JENKINS_CRED, verifyDeployments: false])
        }
      }
    }
  }
}
