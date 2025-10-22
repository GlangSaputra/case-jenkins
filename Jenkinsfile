pipeline {
  agent any

  environment {
    IMAGE = "galanglang/demo-app"
    TAG = "latest"
    DOCKER_CRED = "dockerhub-credential"
    KUBECONFIG_CRED = "kubeconfig-dev"
    NAMESPACE = "default"
    HELM_RELEASE = "casestudy-jenkins1"
  }

  stages {
    stage('Checkout Source Code') {
      steps {
        git url: 'https://github.com/GlangSaputra/case-jenkins.git', branch: 'main'
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          echo "🛠️ Building image ${IMAGE}:${TAG}..."
          bat "docker build -t ${IMAGE}:${TAG} ."
        }
      }
    }

    stage('Push Docker Image') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: "${DOCKER_CRED}",
          usernameVariable: 'USER',
          passwordVariable: 'PASS'
        )]) {
          script {
            echo "📦 Pushing image to DockerHub..."
            bat """
              echo %PASS% | docker login -u %USER% --password-stdin
              docker push ${IMAGE}:${TAG}
            """
          }
        }
      }
    }

    stage('Deploy to Kubernetes (Helm)') {
      steps {
        withCredentials([file(credentialsId: "${KUBECONFIG_CRED}", variable: 'KUBE_FILE')]) {
          script {
            echo "🚀 Deploying to Kubernetes via Helm..."
            bat """
              set KUBECONFIG=%KUBE_FILE%
              helm upgrade --install %HELM_RELEASE% ./helm ^
                --set image.repository=%IMAGE% ^
                --set image.tag=%TAG% ^
                --namespace %NAMESPACE% --create-namespace
            """
          }
        }
      }
    }

    stage('Validate Deployment') {
      steps {
        bat "kubectl get pods --namespace %NAMESPACE%"
        bat "curl -s %MINIKUBE_URL%"
      }
    }
  }

  post {
    success {
      echo "✅ Pipeline Sukses: Aplikasi berhasil dideploy ke Kubernetes"
    }
    failure {
      echo "❌ Pipeline Gagal: Cek log untuk mengetahui error"
    }
  }
}
