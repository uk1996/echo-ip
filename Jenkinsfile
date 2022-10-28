pipeline {
  environment {
    PROJECT = "phonic-realm-360311"
    APP_NAME = "gceme"
    FE_SVC_NAME = "${APP_NAME}-frontend"
    CLUSTER = "jenkins-cd"
    CLUSTER_ZONE = "asia-northeast3-a"
    IMAGE_TAG = "${env.BUILD_NUMBER}"
    BRANCH_NAME = "${env.BRANCH_NAME}"
  }
  
  agent {
    kubernetes {
      yaml '''
      apiVersion: v1
      kind: Pod
      metadata:
        labels:
          app: blue-green-deploy
        name: blue-green-deploy
      spec:
        containers:
        - name: kustomize
          image: sysnet4admin/kustomize:3.6.1
          tty: true
          volumeMounts:
          - mountPath: /usr/bin/kubectl
            name: kubectl
          - mountPath: ~/.kube/config
            name: kube-config
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
        - name: kubectl
          hostPath:
            path: /usr/bin/kubectl
        - name: docker-sock
          hostPath:
            path: /var/run/docker.sock
        - name: kube-config
          hostPath:
            path: ~/.kube/config
      '''
    }
  }
    
  stages {
    stage('git scm update') {
      
      steps {
        git url: 'https://github.com/uk1996/echo-ip.git', branch: 'main'
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
              docker build -t asia-northeast3-docker.pkg.dev/phonic-realm-360311/test-img-registry/quickstart-image:${BRANCH_NAME}_${IMAGE_TAG} .
              docker push asia-northeast3-docker.pkg.dev/phonic-realm-360311/test-img-registry/quickstart-image:${BRANCH_NAME}_${IMAGE_TAG}
              '''
        }
      }
    }
    stage('deploy kubernetes') {
      steps {
        container('kustomize') {
          sh '''
          kubectl config use-context tilda-saas-dev
          kustomize create --resources ./deployment.yaml
          kustomize edit set namesuffix -- -${BRANCH_NAME}
          kustomize edit set image asia-northeast3-docker.pkg.dev/phonic-realm-360311/test-img-registry/quickstart-image:${BRANCH_NAME}_${IMAGE_TAG}
          kustomize build . | kubectl apply -f - --record
          '''
        }
      }
    }
  }
}
