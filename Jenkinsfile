pipeline {
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        metadata:
          labels:
            app: test
        spec:
          containers:
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
  environment{
    DOCKERHUB_USERNAME = "shtlamrut"
    APP_NAME = "kaniko-webapp-demo"
    IMAGE_NAME = "${DOCKERHUB_USERNAME}" + "/" + "${APP_NAME}"
    IMAGE_TAG = "${BUILD_NUMBER}"
  }
 stages {
    stage('Checkout SCM') {
      steps {
        container('git') {
          git url: 'https://github.com/shtlamrut/argocd-demo-deploy.git',
          branch: 'master'
        }
      }
    }
    stage('Build SW'){
      steps {
        container('maven'){
          sh 'mvn -Dmaven.test.failure.ignore=true clean package'
        }
      }
    }
    stage('Build Container Image'){
      steps {
        container('kaniko'){
          sh "/kaniko/executor --context $WORKSPACE --destination $IMAGE_NAME:$IMAGE_TAG"
        }
      }
    }
  }
}
