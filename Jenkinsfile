pipeline {
  agent {
    kubernetes {
      label 'jenkins-slave'
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
        kind: Pod
        metadata:
          labels:
            app: test
        spec:
          containers:
          - name: git
            image: bitnami/git:latest
            command:
            - cat
            tty: true
          - name: kaniko
            image: gcr.io/kaniko-project/executor:debug
            command:
            - cat
            tty: true
            volumeMounts:
            - name: kaniko-secret
              mountPath: /kaniko/.docker
          volumes:
          - name: kaniko-secret
            secret:
              secretName: regcred
              items:
                - key: .dockerconfigjson
                  path: config.json
      '''
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
