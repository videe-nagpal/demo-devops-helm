podTemplate(label: 'mypod', serviceAccount: 'jenkins-tool', containers: [ 
    containerTemplate(
      name: 'docker', 
      image: 'docker', 
      command: 'cat', 
      resourceRequestCpu: '100m',
      resourceLimitCpu: '300m',
      resourceRequestMemory: '300Mi',
      resourceLimitMemory: '500Mi',
      ttyEnabled: true
    ),
    containerTemplate(
      name: 'kubectl', 
      image: 'amaceog/kubectl',
      resourceRequestCpu: '100m',
      resourceLimitCpu: '300m',
      resourceRequestMemory: '300Mi',
      resourceLimitMemory: '500Mi', 
      ttyEnabled: true, 
      command: 'cat'
    ),
    containerTemplate(
      name: 'helm', 
      image: 'alpine/helm:3.12.0', 
      resourceRequestCpu: '100m',
      resourceLimitCpu: '300m',
      resourceRequestMemory: '300Mi',
      resourceLimitMemory: '500Mi',
      ttyEnabled: true, 
      command: 'cat'
    )
  ],

  volumes: [
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
  ]
  ) {
    node('mypod') {

        def REPOSITORY_URI = "vnagpal/demo-devops-helm"
        def HELM_APP_NAME = "nodejs-app-2"
        def HELM_CHART_DIRECTORY = "k8s/nodejs-k8s-cicd"

        stage('Get latest version of code') {
          checkout scm
        }
        stage('Check running containers') {
            container('docker') {  
                sh 'hostname'
                sh 'hostname -i' 
                sh 'docker ps'
                sh 'ls'
            }
            container('kubectl') { 
                sh 'kubectl get pods -n default'
                sh 'if (kubectl get ns demo); then echo ns exists; else kubectl create ns demo; kubectl label namespace demo istio-injection=enabled; fi'
            }
            container('helm') { 
                sh 'helm list'     
            }
        }  

        stage('Build Image'){
            container('docker'){

              withCredentials([usernamePassword(credentialsId: 'docker-login', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                sh 'docker login --username="${USERNAME}" --password="${PASSWORD}"'
                sh "docker build -t ${REPOSITORY_URI}:${BUILD_NUMBER} . --platform linux/x86_64"
                sh 'docker image ls' 
              } 
                
            }
        } 

        stage('Testing') {
            container('docker') { 
              sh 'whoami'
              sh 'hostname -i' 
              sh "docker run ${REPOSITORY_URI}:${BUILD_NUMBER} npm run test "                 
            }
        }

        stage('Push Image'){
            container('docker'){
              withCredentials([usernamePassword(credentialsId: 'docker-login', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                sh 'docker image ls'
                sh "docker push ${REPOSITORY_URI}:${BUILD_NUMBER}"
              }                 
            }
        }
        
        stage('Deploy Image to k8s'){
            container('helm'){
                sh 'helm list'
                sh "helm lint ./${HELM_CHART_DIRECTORY}"
                sh "helm install --wait --timeout=1h --set=image.tag=${BUILD_NUMBER} ${HELM_APP_NAME} ./${HELM_CHART_DIRECTORY} -n demo"
                sh "helm list | grep ${HELM_APP_NAME}"
            }
        }    
        
    }
}
