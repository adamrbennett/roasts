node {
  git 'git@github.com:adamrbennett/roasts.git'

  def img
  stage('Build') {
    img = docker.build("pointsource/roasts:${env.BUILD_NUMBER}")
  }

  docker.withRegistry('https://index.docker.io/v2/', 'docker-registry-credentials') {
    stage('Publish') {
      img.push()
      img.push('latest')
    }
  }

  stage('Deploy') {
    withEnv(['DOCKER_HOST=tcp://mgr1.node.consul:2375']) {
      sh "docker service create --with-registry-auth --name roasts-${env.BUILD_NUMBER} --network sfi -e SERVICE_NAME=roasts -e SERVICE_TAGS=${env.BUILD_NUMBER} pointsource/roasts:${env.BUILD_NUMBER}"
    }
  }
}
