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
    node{
      stage('deploy kubernetes') {
        steps {
          container('kustomize') {
            withKubeConfig([credentialsId: 'LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUVMRENDQXBTZ0F3SUJBZ0lRQ2ZpQlAvb2k5VGxIY3M0aFdFd2pzekFOQmdrcWhraUc5dzBCQVFzRkFEQXYKTVMwd0t3WURWUVFERXlSbE5USTJabUk0WVMxa1pUWTNMVFJoT0dZdFltVXdZeTFtTkRJd1ltSXhNbUk1TTJNdwpJQmNOTWpJeE1ESTFNRGd3TWpNMVdoZ1BNakExTWpFd01UY3dPVEF5TXpWYU1DOHhMVEFyQmdOVkJBTVRKR1UxCk1qWm1ZamhoTFdSbE5qY3ROR0U0WmkxaVpUQmpMV1kwTWpCaVlqRXlZamt6WXpDQ0FhSXdEUVlKS29aSWh2Y04KQVFFQkJRQURnZ0dQQURDQ0FZb0NnZ0dCQUt4aVN3aWFGSnNPckttVVBpWjQ4UU5qQjIybmZwamZaZVVWL1E0cApGbHUwZW8rNGlkREhTRDFhMVBxYlRzd2haVHJTTWdHaWFORWxZV0lNMFZiU1JrZ1drVEJ5d3BhanFDbnpMc0RvCi9zTk15ZTIycnhIK3VaSldOcnN1VHNna0FLQnJPdlRoZUNXR3RsNmFmZ25GUEY3N0x3RmdacXdTN2lXY3BtWncKRk1zKzVManJhVmUvbFF6c3IxOHcySlJTS1FsM3JpNkdmd29tcDlmZmRBbFlrSFpSeGlCSmFOek1LV1JSbTB3UQpyR0RJVDJ6RUZCeTVhVTZOdTNrYUI4NVpJZW5BSmMyVXczUWoydkI1UkxqaUFIVjQxa0ovSFlveE1kcVhOMGZ1Cml3aEw0Rkt3Y2xLaGQxakZiRytwaDVOUUtCWmR1TkRpQ0FXeXl5dTFJbll4YzM0T3Z6U1Jibzl3SnV3SDNwMEgKUXpRaFkvSjBvTUpZVVdtQ3E0bG9MZTJkdVpYbEdQaXVFcHYxVDJ4SUkwY1k5amk2bHI1N3lJdXB5d29VN0ptQQp1OS84dVQva3BJUDR1dC81WWFOWmFLQXBweWZZeEJ3Nnp1R04veWhBbytIdFlyRUs1T2VGaEFtUmV1c2FOakRLClpEMlBHOXFQN1AyRFNZa1c3d0pMTDZZSGl3SURBUUFCbzBJd1FEQU9CZ05WSFE4QkFmOEVCQU1DQWdRd0R3WUQKVlIwVEFRSC9CQVV3QXdFQi96QWRCZ05WSFE0RUZnUVVhKytxL3ZPazUwK0owRGVvQUdic1lnaElML2d3RFFZSgpLb1pJaHZjTkFRRUxCUUFEZ2dHQkFKOU5USWowalVVVUQvajFRWFVabEd5MHpiUWVoM2ZLYmtoTXZ3ZVVSRU1uClVZVEhWQmt3Uy85WSswWk1MMzYxejljekRUb1hTSStwWWtvcXg0NUlXUVdnT1VXK0NCZUZHY3d3MjE1N3NqRmMKd0dhS08ySlpXR1JPVjZUM0htSlpoM0k0SmQrRUJKdVZMdnhuM1ZLdTU5Y0hKQ0wwQ2RkR01uVS9XNlBmUmlFYgpGbEwzeCtmcVJJblZ3dzVVc0hwd3FPMTVpNndheFpGQmhFRTRRNDBMVnNtVkw4ZGFWRjhyaEl4SDJ6UzlQaHJ6CnoyS3dkdzA2aHdMb085YjFYZEUycHQ4L1YySEtySHBWNlppdHB1and6NW9ZQmQ5QmU2NHJkbXdQS0FseGlVaU0KNGdqTVpPbXNJUm13eVJIejVUSHBZKzJFOWVic1F1VUdGMm5TRkFGaUJoSjNuOFVIZFYzMUViUXI1TUpGeHBDNApIS3lmdk9oRzFzZnR0aFpJR1oxVHl6byt1bElyR05zWUU3L28yaXFkU21GR3VJNENXTHNGOU9uOU1RWFpKMk8wClNQb2pIT0wvSk9oVVV3UWNZUkZTVzlqMnUxeU1EK0pEWU44Uy9ET1RhVTRMdE9yQzl5Z0xicUdiNFovelR6M20KWnpmTGNqa2VIQ2l1RXpjWU56ZGdKZz09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K', serverUrl: 'https://34.64.160.246'])
            sh '''
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
}
