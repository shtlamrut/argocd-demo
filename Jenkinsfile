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
        image: gcr.io/kaniko-project/executor:debug
        command:
        - sleep
        args:
        - 9999999
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

    stage('Image Build') {
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

    stage('Deploy to qa') {
      environment {
        GIT_CREDS = credentials('github')
      }
      steps {
        container('tools') {
            sh "git clone https://$GIT_CREDS_USR:$GIT_CREDS_PSW@github.com/shtlamrut/argocd-demo-deploy.git"
            sh "git config --global user.email shtlamrut@gmail.com"
            sh "git config --global user.name shtlamrut"

            dir("argocd-demo-deploy") {
            sh "cd ./overlays/qa && kustomize edit set image my-app=shtlamrut/argocd-demo:${env.GIT_COMMIT}"
            sh "git commit -am 'Update app image tag to ${env.GIT_COMMIT}'"
            sh "git push"
            }    
        }
     }
   }
   stage('Deploy to prod') {
      steps {
        input message:'Approve deployment?'
        container('tools') {
            dir("argocd-demo-deploy") {
            sh "cd ./overlays/prod && kustomize edit set image my-app=shtlamrut/argocd-demo:${env.GIT_COMMIT}"
            sh "git commit -am 'Update app image tag to ${env.GIT_COMMIT}'"
            sh "git push"
            }    
        }
      }
   }
}
}
