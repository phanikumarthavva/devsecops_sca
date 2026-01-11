pipeline {
  agent any

  environment {
    IMAGE_REPO = "docker.io/phanikumart/devsecops"
    IMAGE_TAG  = "${env.BUILD_NUMBER}"
    DOCKER_CRED = "dockerhub-creds"
    //KUBECONFIG_CRED = "do-kubeconfig"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build') {
      steps {
        sh """
          docker build -t ${IMAGE_REPO}:${IMAGE_TAG} .
          docker tag ${IMAGE_REPO}:${IMAGE_TAG} ${IMAGE_REPO}:latest
        """
      }
    }

    stage('Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKER_CRED}", usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
          sh """
            echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
            docker push ${IMAGE_REPO}:${IMAGE_TAG}
            docker push ${IMAGE_REPO}:latest
          """
        }
      }
    }
	stage('OWASP Dependency-Check Vulnerabilities') {
      steps {
        dependencyCheck additionalArguments: ''' 
                    -o './'
                    -s './'
                    -f 'ALL' 
                    --prettyPrint''', odcInstallation: 'OWASP Dependency-Check Vulnerabilities'
        
        dependencyCheckPublisher pattern: 'dependency-check-report.xml'
      }
    }
    /*
    stage('Deploy to DO Kubernetes') {
      steps {
        withCredentials([file(credentialsId: "${KUBECONFIG_CRED}", variable: 'KUBECONFIG_FILE')]) {
          sh """
            export KUBECONFIG="$KUBECONFIG_FILE"

            # Update the image in the manifest to the build tag, Working copy
            sed -i 's|image:.*|image: ${IMAGE_REPO}:${IMAGE_TAG}|g' k8s/deployment.yaml

            kubectl apply -f k8s/
            kubectl rollout status deployment/dep-devsecops
            kubectl get svc svc-devsecops
          """
        }
      }
    }
	*/
  }
}
