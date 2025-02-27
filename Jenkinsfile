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
                image: docker:latest
                command:
                - cat
                tty: true
                volumeMounts:
                  - mountPath: /var/run/docker.sock
                    name: docker-sock
              - name: envstr
                image: bhgedigital/envsubst
                command:
                - cat
                tty: true
              - name: kubectl
                image: bitnami/kubectl
                command:
                - cat
                tty: true
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
//       stage('Docker Install') {
//          steps {
//             container('docker'){
//                sh 'apt-get update && apt-get install && ca-certificates && curl && gnupg && lsb-release'
//                sh 'mkdir -p /etc/apt/keyrings'
//                sh 'curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg'
//             }
//          }
//       }
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

      stage('save env for Cluster') {
          steps {
            container('envstr'){
               sh 'envsubst < ${WORKSPACE}/deploy.yaml | ls | cat deploy.yaml'
            }
          }
      }
      stage('Deploy to Cluster') {
          steps {
            container('kubectl'){
               sh 'kubectl apply -f -'
            }
          }
      }
   }
}
