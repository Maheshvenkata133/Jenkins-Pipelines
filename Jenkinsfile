pipeline {
  agent any 

  environment {
    REPO_URL = 'https://github.com/user/app-repo.git'  //Application repo 
    GITOPS_REPO_URL ='https://github.com/user/gitops-repo.git'   //Gitops repo 
    BRANCH = 'main'
    DOCKER IMAGE ='harbor.example.com/project/app'    //change as needed
    IMAGE_TAG = "v$(BUILD_NUMBER)"   //versioned tag
    K8S_MANIFESTS_PATH ='k8s/deployment.yaml'    //k8s manifest location
    ARGOCD_APP_NAME ='my-app'
    ARGOCD_SERVER ='argocd.prod.com'
    ARGOCD_AUTH_TOKER = credentials('argocd-token')    //argocd authentication token
  }

  triggers {
    githubPush()     //triggers when commit is pushed
  }

  stages {
    stage ('clone repository') {
      steps {
        git branch = "${BRANCH}", "${REPO_URL}" 
      }
    }
    stage ('Build docker image') {
      steps {
        script {
          sh "docker login harbor.example.com -u user -password"     //use credentials store for security
          sh "docker push ${DOCKER_IMAGE}:${IMAGE_TAG}"
        }
      }
    }
    stage ('Update k8s manifest') {
      steps {
        script {
          sh '''
          git clone ${GITOPS_REPO_URL} gitops
          cd gitops


          ## Update the image tag in the k8s manifest
          sed -i "s|image: ${DOCKER_IMAGE}.*|image: ${DOCKER_IMAGE}:${DOCKER_TAG}|" ${K8S_MANIFEST_PATH}

          git config user.name "jenkins"
          git config user.email "jenkins@example.com"
          git add ${K8S_MANIFEST_PATH}
          git commit -m "update deployment image to ${IMAGE_TAG}
          git push origin ${BRANCH}
          '''
        }
      }
    }
    stage ("Trigger ArgoCD Sync") {
      steps {
        script {
          sh '''
          cult -X POST -H "Authorization: Bearer ${ARGOCED_AUTH_TOKEN}" \
          -H "Content-Type: Application/json" \
          "https://{ARGOCD+_SERVER}api/v1/applications/${ARGOCD_APP_NAME}/sync"
          '''
        }
      }
    }
  }

  post {
    success {
      echo "Build,push and deployment successfull!"
    }
    failure {
      echo "Pipeline failed, check logs"
    }
  }
}
  










    
  






  
      
      
