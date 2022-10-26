pipeline {
  environment {
    PROJECT = "phonic-realm-360311"
    APP_NAME = "gceme"
    FE_SVC_NAME = "${APP_NAME}-frontend"
    CLUSTER = "jenkins-cd"
    CLUSTER_ZONE = "asia-northeast3-a"
    IMAGE_TAG = "${env.BUILD_NUMBER}"
    JENKINS_CRED = "${env.GCLOUD_TOKEN}"
    
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
    stage('docker build and push') {
      steps {
         container('docker'){
              sh '''
              docker login -u oauth2accesstoken -p ${JENKINS_CRED} https://asia-northeast3-docker.pkg.dev
              docker build -t asia-northeast3-docker.pkg.dev/phonic-realm-360311/test-img-registry/quickstart-image:${IMAGE_TAG} .
              docker push asia-northeast3-docker.pkg.dev/phonic-realm-360311/test-img-registry/quickstart-image:${IMAGE_TAG}
              '''
        }
      }
    }
    stage('deploy kubernetes') {
      steps {
        container('kustomize') {
          sh '''
          kubectl create deployment pl-bulk-prod --image=asia-northeast3-docker.pkg.dev/phonic-realm-360311/test-img-registry/quickstart-image:${IMAGE_TAG}
          kubectl expose deployment pl-bulk-prod --type=LoadBalancer --port=8080 \
                                                 --target-port=80 --name=pl-bulk-prod-svc
          '''
        }
      }
    }
  }
}
