pipeline {
  agent none
  stages {
    stage('Build') {
      agent {
        docker {
          image 'python:3.11.5-alpine3.18'
        }

      }
      steps {
        sh 'python -m py_compile sources/add2vals.py sources/calc.py'
        stash(name: 'compiled-results', includes: 'sources/*.py*')
      }
    }

    stage('Test') {
      agent {
        docker {
          image 'qnib/pytest'
        }

      }
      post {
        always {
          junit 'test-reports/results.xml'
        }

      }
      steps {
        sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
      }
    }

    stage('Deliver') {
      agent any
      environment {
        VOLUME = '$(pwd)/sources:/src'
        IMAGE = 'cdrx/pyinstaller-linux:python2'
      }
      post {
        success {
          archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
          sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
        }

      }
      steps {
        dir(path: env.BUILD_ID) {
          unstash 'compiled-results'
          sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
        }

      }
    }

  }
  options {
    skipStagesAfterUnstable()
  }
}