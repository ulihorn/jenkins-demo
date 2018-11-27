throttle(['throttleDocker']) {
  node('docker') {
    wrap([$class: 'AnsiColorBuildWrapper']) {
      try{
        stage('Setup') {
          checkout scm
          sh '''
            ./ci/docker-down.sh
            ./ci/docker-up.sh
          '''
        }
        stage('Test'){
          parallel (
            "unit": {
              sh '''
                ./ci/test/unit.sh
              '''
            },
            "functional": {
              sh '''
                ./ci/test/functional.sh
              '''
            }
          )
        }
        stage('Capacity Test') {
          sh '''
            ./ci/test/stress.sh
          '''
        }
        stage('Deploy to Docker Swarm') {
          sh '''
            version=$(date +%Y%m%d%H%M)
            ./cd/publish.sh alpha registry:443 $version
            export DOCKER_HOST="tcp://10.180.252.116:2375"
            ./cd/deploy-swarm.sh alpha registry:443 $version
          '''
        }
      }
      finally {
        stage('Cleanup') {
          sh '''
            ./ci/docker-down.sh
          '''
        }
      }
    }
  }
}
