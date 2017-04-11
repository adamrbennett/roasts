node {
  git 'git@github.com:adamrbennett/jenkins-docker.git'

  def nodejs = docker.image('node:7.4');

  stage('Unit Test') {
    nodejs.inside {
      sh 'npm -q install --prefix app'
      sh 'npm --prefix app test'
    }
    junit 'app/test-results.xml'
    publishHTML (target: [
      allowMissing: false,
      alwaysLinkToLastBuild: false,
      keepAll: false,
      reportDir: 'app/coverage',
      reportFiles: 'index.html',
      reportName: "Code Coverage Report"
    ])
  }

  stage('Acceptance Test') {
    nodejs.inside {
      sh 'npm -q install --prefix api'
      sh './api/node_modules/cucumber/bin/cucumber.js ./api/features --format=json > ./api/acceptance-test-results.json'
      sh 'node ./api/test-report.js'
    }
    publishHTML (target: [
      allowMissing: false,
      alwaysLinkToLastBuild: false,
      keepAll: false,
      reportDir: 'api',
      reportFiles: 'acceptance-test-results.html',
      reportName: "Acceptance Test Results"
    ])
  }

  def appImg
  def apiImg
  stage('Build') {
    appImg = docker.build("localhost:5000/jenkins-docker-app:${env.BUILD_NUMBER}", '-f app/Dockerfile app')
    apiImg = docker.build("localhost:5000/jenkins-docker-api:${env.BUILD_NUMBER}", '-f api/Dockerfile api')
  }

  stage('Integration Test') {
    apiImg.withRun('-p 3000:80') {
      nodejs.inside {
        sh 'npm -q install --prefix api'
        sh "API_ROOT='http://172.17.0.1:3000' TEST_DIR=./api ./api/node_modules/jenkins-mocha/bin/jenkins.js ./api/test --no-coverage"
      }
      junit 'api/xunit.xml'
    }
  }

  docker.withRegistry('http://localhost:5000/') {
    stage('Publish') {
      appImg.push()
      apiImg.push()

      appImg.push('latest')
      apiImg.push('latest')
    }
  }
}
