pipeline {
    environment {
      DOCKER = credentials('docker-hub')
    }
  agent any
  stages {
// Building your Test Images
    stage('BUILD') {
      parallel {
        stage('Express Image') {
          steps {
            sh 'sudo docker build -f express-image/Dockerfile \
            -t nodeapp-dev:trunk .'
          }
        }
        stage('Test-Unit Image') {
          steps {
            sh 'sudo docker build -f test-image/Dockerfile \
            -t test-image:latest .'
          }
        }
      }
      post {
        failure {
            echo 'This build has failed. See logs for details.'
        }
      }
    }
// Performing Software Tests
    stage('TEST') {
      parallel {
        stage('Mocha Tests') {
          steps {
            sh 'sudo docker run --name nodeapp-dev -d \
            -p 9000:9000 nodeapp-dev:trunk'
            sh 'sudo docker run --name test-image -v $PWD:/JUnit \
            --link=nodeapp-dev -d -p 9001:9000 \
            test-image:latest'
          }
        }
        stage('Quality Tests') {
          steps {
            sh 'sudo docker login --username $DOCKER_USR --password $DOCKER_PSW'
            sh 'sudo docker tag nodeapp-dev:trunk keshav47/nodeapp-dev:latest'
            sh 'sudo docker push keshav47/nodeapp-dev:latest'
          }
        }
      }
      post {
        success {
            echo 'Build succeeded.'
        }
        unstable {
            echo 'This build returned an unstable status.'
        }
        failure {
            echo 'This build has failed. See logs for details.'
        }
      }
    }
// Deploying your Software
    stage('DEPLOY') {
          when {
           branch 'main'  //only run these steps on the master branch
          }
            steps {
                    retry(3) {
                        timeout(time:10, unit: 'MINUTES') {
                            sh 'sudo docker tag nodeapp-dev:trunk keshav47/nodeapp-prod:latest'
                            sh 'sudo docker push keshav47/nodeapp-prod:latest'
                            sh 'sudo docker save keshav47/nodeapp-prod:latest | gzip > nodeapp-prod-golden.tar.gz'
                        }
                    }

            }
            post {
                failure {
                    sh 'sudo docker stop nodeapp-dev test-image'
                    sh 'sudo docker system prune -f'
                    deleteDir()
                }
            }
    }
// JUnit reports and artifacts saving
    stage('REPORTS') {
      steps {
        junit 'reports.xml'
        archiveArtifacts(artifacts: 'reports.xml', allowEmptyArchive: true)
        archiveArtifacts(artifacts: 'nodeapp-prod-golden.tar.gz', allowEmptyArchive: true)
      }
    }
// Doing containers clean-up to avoid conflicts in future builds
    stage('CLEAN-UP') {
      steps {
        sh 'sudo docker stop nodeapp-dev test-image'
        sh 'sudo docker system prune -f'
        deleteDir()
      }
    }
  }
}

