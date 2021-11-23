node {
  def commit_id
  stage('set vars') {
    checkout scm
    sh "git rev-parse --short HEAD > .git/commit-id"
    commit_id = readFile('.git/commit-id').trim()
  }
  stage('test') {
    nodejs(nodeJSInstallationName: 'nodejs') {
      sh 'npm install'
      sh 'npm test'
    }
  }
  stage('docker build/push') {
    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
      def app = docker.build("or11/react-test-app:${commit_id}", '.').push()
    }
  }
  stage('deploy') {
    try {
        sh '''
            docker stop react-test-app
            docker rm react-test-app
        '''
    } catch (err) {
        echo: 'caught error: $err'
    }
    sh "docker run --restart=always --name react-test-app -p 3000:3000 -d or11/react-test-app:${commit_id}"
  }
}
