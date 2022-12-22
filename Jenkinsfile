pipeline {
  agent {
    kubernetes {
      label 'jenkins-slave'
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1 
kind: Pod 
metadata: 
    name: dind 
spec: 
    containers: 
      - name: docker-cmds 
        image: docker:1.12.6 
        command: ['docker', 'run', '-p', '80:80', 'httpd:latest'] 
        resources: 
            requests: 
                cpu: 10m 
                memory: 256Mi 
        env: 
          - name: DOCKER_HOST 
            value: 127.0.0.1
      - name: dind-daemon 
        image: docker:1.12.6-dind 
        resources: 
            requests: 
                cpu: 20m 
                memory: 512Mi 
        securityContext: 
            privileged: true 
        volumeMounts: 
          - name: docker-graph-storage 
            mountPath: /var/lib/docker 
    volumes: 
      - name: docker-graph-storage 
        emptyDir: {}
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

