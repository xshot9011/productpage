def scmVars

pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: docker
    image: docker:20.10.3-dind
    command:
    - dockerd
    - --host=unix:///var/run/docker.sock
    - --host=tcp://0.0.0.0:2375
    - --storage-driver=overlay2
    tty: true
    securityContext:
        privileged: true
  - name: helm
    image: lachlanevenson/k8s-helm:v3.5.0
    command:
    - cat
    tty: true
"""
        }
    }
    
	environment {
        ENV_NAME = "${BRANCH_NAME == "master" ? "uat" : "${BRANCH_NAME}"}"
    }
	
    stages {
        
        stage('Clone ratings source code') {
            steps {
                container('jnlp') {
                    script {
                        scmVars = git branch: "${BRANCH_NAME}",
                                      credentialsId: 'bookinfo-git-deploy-key',
                                      url: 'git@gitlab.com:workshop35/bookinfo/productpage.git'
                    }
                }
            }
        }
		
		stage('Build ratings Docker Image and push') {
            steps {
                container('docker') {
                    script {
                        docker.withRegistry('', 'registry-bookinfo') {
                            docker.build('bigdev2000/productpage:${ENV_NAME}').push()
                        }
                    }
                }
            }
        }
		
        stage('Deploy with Helm') {
            steps {
                container('helm') {
                    script {
						withKubeConfig([credentialsId: 'gke-opsta-cluster-sa-secret-file']) {
							sh "helm upgrade -f k8s/helm-values/values-bookinfo-${ENV_NAME}-productpage.yaml --wait \
                                 --set extraEnv.COMMIT_ID=${scmVars.GIT_COMMIT} \
                                 --namespace opsta-${ENV_NAME} bookinfo-${ENV_NAME}-productpage k8s/helm/ \
                                 --install"
						}
                    }
                }
            }
        }
    }
}
