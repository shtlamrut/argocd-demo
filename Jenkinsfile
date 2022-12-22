pipeline {
  agent {
    kubernetes {
      label 'jenkins-slave'
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:latest
    args:
    - "--context=git://<Git API token>@github.com/computingforgeeks/kubernetes-kaniko.git #refs/heads/master"
    - "--destination=shtlamrut/kaniko-demo-image:1.0"
    volumeMounts:
    - name: kaniko-secret
      mountPath: /kaniko/.docker
  restartPolicy: Never
  volumes:
  - name: kaniko-secret
    secret:
      secretName: dockercred
      items:
        - key: .dockerconfigjson
          path: config.json
  - name: docker
    env:
    - name: DOCKER_HOST
      value: 127.0.0.1
    image: docker
    command:
    - cat
    tty: true
  - name: tools
    image: nekottyo/kustomize-kubeval
    command:
    - cat
    tty: true  
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
