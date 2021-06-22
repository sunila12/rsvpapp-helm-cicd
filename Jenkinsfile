pipeline {
    agent {
      kubernetes  {
            label 'jenkins-slave'
             defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: dind
    image: docker:18.09-dind
    securityContext:
      privileged: true
  - name: docker
    env:
    - name: DOCKER_HOST
      value: 127.0.0.1
    image: docker:18.09
    command:
    - cat
    tty: true
  - name: tools
    image: argoproj/argo-cd-ci-builder:v1.0.0
    command:
    - cat
    tty: true
"""
        }
    }
  environment {
      IMAGE_REPO = "hemanthhr/rsvpdummy"
      // Instead of nkhare, use your repo name
  }
  stages {
    stage('Build') {
      environment {
        DOCKERHUB_CREDS = credentials('dockerhub')
      }
      steps {
        container('docker') {
          // Build new image
          sh "until docker image ls; do sleep 3; done && docker image build -t  ${env.IMAGE_REPO}:${env.GIT_COMMIT} ."
          // Publish new image
          sh "docker login --username $DOCKERHUB_CREDS_USR --password $DOCKERHUB_CREDS_PSW && docker image push ${env.IMAGE_REPO}:${env.GIT_COMMIT}"
        }
      }
    }
    stage('Deploy to staging') {
      environment {
        GIT_CREDS = credentials('github')
        GIT_REPO_URL = "github.com/hemanth344/rsvpapp-helm-cicd.git"
        GIT_REPO_EMAIL = 'hemanth@cloudyuga.guru'
        GIT_REPO_BRANCH = "master"
       // Update above variables with your user details
      }
      steps {
        container('tools') {
            sh "git clone https://$GIT_CREDS_USR:$GIT_CREDS_PSW@${env.GIT_REPO_URL}"
            sh "git config --global user.email ${env.GIT_REPO_EMAIL}"
          dir("rsvpapp-helm-cicd") {
              sh "git checkout ${env.GIT_REPO_BRANCH}"
              sh "wget https://github.com/mikefarah/yq/releases/download/v4.2.0/yq_linux_amd64.tar.gz"
              sh "tar xvf yq_linux_amd64.tar.gz"
              sh "mv yq_linux_amd64 /usr/bin/yq"
              sh "cd ./package" 
              sh "yq --version"
              steps{
              try{
                  sh "cd package && myenv="${env.IMAGE_REPO}" yq eval --null-input '.image.tag = strenv(myenv)' -i dummy.yaml"
                  sh "cat dummy.yaml"
              }catch (err) {
                    echo: 'caught error: $err'
                }
              }
              sh "yq --version"
            sh "git commit -am 'Publish new version' && git push || echo 'no changes'"
          }
        }
      }
    }   
  }
}
