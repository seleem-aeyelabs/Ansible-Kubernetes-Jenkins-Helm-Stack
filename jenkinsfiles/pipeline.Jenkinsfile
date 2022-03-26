pipeline {
  environment {
       registry = ""
       registryCredential = ''
       dockerImage = ''
       BUILD_NUMBER = ''
       k8s_cluster_url = ''
       k8s_Credential = ''
   }
   agent {
        kubernetes {
            cloud 'k8s-cluster-01'
            yaml '''
              apiVersion: v1
              kind: Pod
              spec:
                tolerations:
                  - key: "Worker"
                    operator: "Equal"
                    value: "True"
                    effect: "NoSchedule"
                containers:
                - name: busybox
                  image: busybox:latest
                  command: ["sleep"]
                  args: ["99d"]
                  securityContext:
                    privileged: true
                - name: docker
                  image: docker:19.03.1-dind
                  command: ["sleep"]
                  args: ["99d"]
                  securityContext:
                    privileged: true
                  env:
                    - name: DOCKER_TLS_CERTDIR
                      value: ""
                  volumeMounts:
                  - name: docker-socket
                    mountPath: /var/run/docker.sock
                - name: helm
                  image: alpine/helm
                  command: ["sleep"]
                  args: ["99d"]
                  securityContext:
                    privileged: true
                volumes:
                - name: docker-socket
                  hostPath:
                    path: /var/run/docker.sock
                    type: Socket
      '''
        }
    }
    stages {
        stage('Apply changes') {
          steps {
            container('busybox') {
              dir("${env.WORKSPACE}/app"){
                git branch: "main", url: 'https://github.com/ASeleem/Ansible-Kubernetes-Jenkins-Helm-Stack.git'
              }
              dir("${env.WORKSPACE}/app/src"){
                script {
                    sh 'chmod +x change.sh'
                    sh './change.sh'
                }
              }
            }
          }
        }

        stage('Building Docker Image') {
          steps {
            container('docker') {
              dir("${env.WORKSPACE}/app/src"){
                script {
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
              }
              script {
                docker.withRegistry( '', registryCredential ) {
                    dockerImage.push()
                }
              }
            }
          }
        }
        stage('Deploy Bulletin Board App ') {
          steps {
            container('helm') {
              dir("${env.WORKSPACE}/app/Helm/bulletinapp"){
                withKubeConfig([credentialsId: k8s_Credential, serverUrl: k8s_cluster_url]) {
                  sh 'helm upgrade --install bulletinapp -n demo .'
                }
              }
            }
          }
        }
    }
}
