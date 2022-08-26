pipeline {
   agent {
       kubernetes {
         yaml '''
            apiVersion: v1
            kind: Pod
            spec:
              serviceAccountName: jenkins-admin
              containers:
              - name: docker
                image: ubuntu:latest
                command:
                - cat
                tty: true
                volumeMounts:
                  - mountPath: /var/run/docker.sock
                    name: docker-sock
              volumes:
              - name: docker-sock
                hostPath:
                  path: /var/run/docker.sock  

            '''
          defaultContainer 'shell'
       }
      
   
   }

   environment {
     // You must set the following environment variables
     // ORGANIZATION_NAME
     // YOUR_DOCKERHUB_USERNAME (it doesn't matter if you don't have one)
     
     SERVICE_NAME = "fleetman-webapp"
     REPOSITORY_TAG="${YOUR_DOCKERHUB_USERNAME}/${ORGANIZATION_NAME}-${SERVICE_NAME}:${BUILD_ID}"
   }

   stages {
      stage('Preparation') {
         steps {
            container('docker'){
               cleanWs()
               git credentialsId: 'GitHub', url: "https://github.com/${ORGANIZATION_NAME}/${SERVICE_NAME}"
            }
         }
      }
      stage('Docker Install') {
         steps {
            container('docker'){
               sh '''
                 apt-get update && apt-get install && ca-certificates && curl && gnupg && lsb-release
                 mkdir -p /etc/apt/keyrings
                 curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
               '''
            }
         }
      }
      stage('Build') {
         steps {
            container('docker'){
               sh 'echo No build required for Webapp.'
            }
         }
      }

      stage('Build and Push Image') {
         steps {
            container('docker'){
               sh 'docker image build -t ${REPOSITORY_TAG} .'
            }
         }
      }

      stage('Deploy to Cluster') {
          steps {
            container('docker'){
               sh '''
               apt add gettext-base
               envsubst < ${WORKSPACE}/deploy.yaml | kubectl apply -f -'
               '''
            }
          }
      }
   }
}
