pipeline {
  agent {
    kubernetes {
      label 'jenkins-slave'
       defaultContainer 'jnlp'
       yaml """
apiVersion: v1
kind: Pod
spec:
  volumes:
    - name: sharedvolume
      emptyDir: {}
    - name: docker-socket
      emptyDir: {}
   containers:
    - name: docker
      env:
    - name: DOCKER_HOST
      value: 127.0.0.1
     image: docker
     command:
      - cat
      tty: true
  volumeMounts:
    - name: docker-socket
      mountPath: /var/run
    - name: sharedvolume
      mountPath: /root/.docker
    - name: docker-daemon
      image: docker:19.03.1-dind
      securityContext:
        privileged: true
  volumeMounts:
    - name: docker-socket
      mountPath: /var/run
    - name: sharedvolume
      mountPath: /root/.docker
    - name: tools
      image: nekottyo/kustomize-kubeval
      command:
      - cat
      tty: true
  serviceAccountName: "jenkins"
"""
}
}
  stages {

    stage('Build') {
      environment {
        DOCKERHUB_CREDS = credentials('dockerHub')
      }
      steps {
        container('docker') {
          sh "docker build -t shtlamrut/argocd-demo:${env.GIT_COMMIT} ."
          sh "docker login --username $DOCKERHUB_CREDS_USR --password $DOCKERHUB_CREDS_PSW" 
          sh "docker push shtlamrut/argocd-demo:${env.GIT_COMMIT}"
        }
      }
    }

    stage('Deploy qa') {
      environment {
        GIT_CREDS = credentials('github')
      }
      steps {
        container('tools') {
          sh"""
            git clone https://$GIT_CREDS_USR:$GIT_CREDS_PSW@github.com/shtlamrut/argocd-demo-deploy.git
            git config --global user.email shtlamrut@gmail.com
            git config --global user.name shtlamrut
            cd ./argocd-demo-deploy/chart
            def text = readFile file: 'values.yaml'
            text = text.replaceAll("%tag%", "${env.GIT_COMMIT}") 
            export GIT_COMMIT=${env.GIT_COMMIT}
            git commit -am 'Update app image tag to ${env.GIT_COMMIT}'
            git push
         """   
        }    
      }
    }
  }
}
