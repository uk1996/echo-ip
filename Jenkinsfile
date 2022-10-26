pipeline {
  environment {
    PROJECT = "phonic-realm-360311"
    APP_NAME = "gceme"
    FE_SVC_NAME = "${APP_NAME}-frontend"
    CLUSTER = "jenkins-cd"
    CLUSTER_ZONE = "asia-northeast3-a"
    IMAGE_TAG = "gcr.io/${PROJECT}/${APP_NAME}:${env.BRANCH_NAME}.${env.BUILD_NUMBER}"
    JENKINS_CRED = "ya29.a0Aa4xrXN8gOlhhM9ky_sPGFv8OhCLlKlMDEOPoCIQcy8XyuOOd08Cq4DsNxVtxEnHtKazntNabJGJ-CcwXS-TvNWmSI2wQQWg4OqZXyiVifay2_brtcO757KNe71IzigtnpctO5wfwOVoAGf4XYa1vtPHWKfIkG95nSFAfpcGbUo0oER-S14r4DVGvoeXbROpjmFYDqLqkFPMmC3-4bSQg-NOggWf9jWT0klzmtonsQH_04O3WQW6hpuYg9HYqwRclb6_HD8aCgYKATASARASFQEjDvL9Q29XpoXuOrS6H51lUt0pPg0270"
    
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
      '''
    }
  }
    
  stages {
    stage('git scm update') {
      
      steps {
        git url: 'https://github.com/uk1996/echo-ip.git', branch: 'main'
      }
    }
//     stage('set auth') {
//       steps {
//          container('gcloud'){
//               sh '''
//               gcloud auth print-access-token | docker login -u oauth2accesstoken --password-stdin https://asia-northeast3-docker.pkg.dev
//               '''
//         }
//       }
//     }
    stage('docker build and push') {
      steps {
         container('docker'){
              sh '''
              docker login -u oauth2accesstoken -p ${JENKINS_CRED} https://asia-northeast3-docker.pkg.dev
              docker build -t asia-northeast3-docker.pkg.dev/phonic-realm-360311/test-img-registry/quickstart-image:tag1 .
              docker push asia-northeast3-docker.pkg.dev/phonic-realm-360311/test-img-registry/quickstart-image:tag1
              '''
        }
      }
    }
    stage('deploy kubernetes') {
      steps {
        container('kustomize') {
          sh '''
          kubectl create deployment pl-bulk-prod --image=asia-northeast3-docker.pkg.dev/phonic-realm-360311/test-img-registry/quickstart-image:tag1
          kubectl expose deployment pl-bulk-prod --type=LoadBalancer --port=8080 \
                                                 --target-port=80 --name=pl-bulk-prod-svc
          '''
        }
      }
    }
  }
}
