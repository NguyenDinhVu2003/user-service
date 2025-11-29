pipeline {
  agent any

  environment {
    DOCKERHUB_CREDENTIALS = credentials('DOCKER_HUB_CREDENTIAL')
    GITHUB_TOKEN = credentials('github-token')
    VERSION = "${env.BUILD_ID}"

  }

  tools {
    maven "Maven"
  }

  stages {

    stage('Maven Build'){
        steps{
        sh 'mvn clean package  -DskipTests'
        }
    }

     stage('Run Tests') {
      steps {
        sh 'mvn test'
      }
    }



      stage('Docker Build and Push') {
      steps {
          sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
          sh 'docker build -t nguyendinhvu/user-service:${VERSION} .'
          sh 'docker push nguyendinhvu/user-service:${VERSION}'
      }
    }


     stage('Cleanup Workspace') {
      steps {
        deleteDir()

      }
    }



    stage('Update Image Tag in GitOps') {
        steps {
            script {
                sh """
                    rm -rf deployment-service
                    git clone https://${GITHUB_TOKEN}@github.com/NguyenDinhVu2003/deployment-service.git
                    cd deployment-service
                    git config user.email "jenkins@ci.com"
                    git config user.name "Jenkins CI"
                    sed -i "s|image:.*|image: nguyendinhvu/user-service:${VERSION}|" aws/user-manifest.yml
                    git add .
                    git commit -m "Update image tag to ${VERSION}"
                    git push
                """
            }
        }
    }

  }

}


