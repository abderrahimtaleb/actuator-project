pipeline {
  agent any


  environment {
    PROJECT_NAME='myproject'
    DOMAIN=’mydomain.com'
    STACK=’mystack'
    DOCKER_REGISTRY=’https://registry.hub.docker.com'
    CONTAINER=’vendor/app'
    VERSION="1.${BUILD_NUMBER}"
  }
  stages {
    stage(’Tag Git Commit’) {
      steps {
        sshagent ([’jenkins’]) {
            script {
                sh '’'
                    git tag -a "v${VERSION}" -m "Jenkins"
                    git push origin "v${VERSION}" -vvv
                '’'
            }
        }
      }
    }
    stage(’Build Image’) {
      steps {
        script {
            docker.withRegistry("${DOCKER_REGISTRY}", 'docker-registry-credentials’) {
                def img = docker.build("${CONTAINER}:${VERSION}")
                img.push()
                sh "docker rmi ${img.id}"
            }
        }
      }
    }
    stage(’Deploy Stack’) {
      steps {
          withCredentials([
              usernamePassword(
                  credentialsId: 'docker-registry-credentials’,
                  usernameVariable: 'DOCKER_USER’,
                  passwordVariable: 'DOCKER_PASSWORD'
              )
          ])
          {
            script {
                echo "Deploying Container Stack to Docker Cluster"
                sh "ansible-playbook -i devops/inventories/manager1/hosts devops/manager1.yml --extra-vars=\"{’WORKSPACE’: '${env.WORKSPACE}’, 'DOMAIN’: '${env.DOMAIN}’, 'PROJECT’: '${env.PROJECT}’, 'STACK’: '${env.STACK}’, 'VERSION’: '${env.VERSION}’, 'DOCKER_REGISTRY’: '${env.DOCKER_REGISTRY}’, 'DOCKER_USER’: '${env.DOCKER_USER}’, 'DOCKER_PASSWORD’: '${env.DOCKER_PASSWORD}’}\" -vvv"
            }
          }
      }
    }
  }
}
