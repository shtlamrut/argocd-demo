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
  - name: tools
    image: nekottyo/kustomize-kubeval
    command:
    - cat
    tty: true  
  - name: docker
    image: docker:19.03.1
    command:
    - sleep
    args:
    - 99d
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
          sh " docker build -t shtlamrut/argocd-demo:${env.GIT_COMMIT} ."
          sh " docker login --username $DOCKERHUB_CREDS_USR --password $DOCKERHUB_CREDS_PSW" 
          sh " docker push shtlamrut/argocd-demo:${env.GIT_COMMIT}"
        }
      }
    }

    stage('Deploy qa') {
      environment {
        GIT_CREDS = credentials('github')
      }
      steps {
        container('tools') {
          sh "git clone https://$GIT_CREDS_USR:$GIT_CREDS_PSW@github.com/shtlamrut/argocd-demo-deploy.git"
          sh "git config --global user.email shtlamrut@gmail.com"
          sh "git config --global user.name shtlamrut"

          dir("argocd-demo-deploy") {
            sh "cd ./overlays/qa && kustomize edit set image my-app=*:${env.GIT_COMMIT}"
            sh "kustomize build ./overlays/qa"
            sh "git commit -am 'Publish new version'"
            sh "git push"
          }
        }    
      }
    }

    stage('Deploy to Prod') {
      steps {
        script {
                def deploymentDelay = input id: 'Deploy', message: 'Deploy to production?', submitter: 'rkivisto,admin', parameters: [choice(choices: ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11', '12', '13', '14', '15', '16', '17', '18', '19', '20', '21', '22', '23', '24'], description: 'Hours to delay deployment?', name: 'deploymentDelay')]
                sleep time: deploymentDelay.toInteger(), unit: 'HOURS'
            }
        container('tools') {
          dir("argocd-demo-deploy") {
            sh "cd ./overlays/prod && kustomize edit set image my-app=*:${env.GIT_COMMIT}"
            sh "kustomize build ./overlays/prod"
            sh "git commit -am 'Publish new version'"  
            sh "git push"
          }
        }  
      }
    }
  }
}
