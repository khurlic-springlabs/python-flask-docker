pipeline {
  agent any
  stages {
    stage('CI Build and push snapshot') {
      when {
        branch 'PR-*'
      }
      environment {
        PREVIEW_VERSION = "0.0.0-SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER"
        PREVIEW_NAMESPACE = "$APP_NAME-$BRANCH_NAME".toLowerCase()
        HELM_RELEASE = "$PREVIEW_NAMESPACE".toLowerCase()
      }
      steps {
        sh 'python -m unittest'
        sh "export VERSION=$PREVIEW_VERSION && skaffold build -f skaffold.yaml"
        sh "jx step post build --image $DOCKER_REGISTRY/$ORG/$APP_NAME:$PREVIEW_VERSION"
        dir(path: './charts/preview') {
          sh 'make preview'
          sh "jx preview --app $APP_NAME --dir ../.."
        }

      }
    }
    stage('Build Release') {
      when {
        branch 'master'
      }
      steps {
        git 'https://github.com/khurlic-springlabs/python-flask-docker.git'
        sh 'echo $(jx-release-version) > VERSION'
        sh 'jx step tag --version $(cat VERSION)'
        sh 'python -m unittest'
        sh 'export VERSION=`cat VERSION` && skaffold build -f skaffold.yaml'
        sh "jx step post build --image $DOCKER_REGISTRY/$ORG/$APP_NAME:\$(cat VERSION)"
      }
    }
    stage('Promote to Environments') {
      when {
        branch 'master'
      }
      steps {
        dir(path: './charts/python-flask-docker') {
          sh 'jx step changelog --version v$(cat ../../VERSION)'
          sh 'jx step helm release'
          sh 'jx promote -b --all-auto --timeout 1h --version $(cat ../../VERSION)'
        }

      }
    }
  }
  environment {
    ORG = 'khurlic'
    APP_NAME = 'python-flask-docker'
    CHARTMUSEUM_CREDS = credentials('jenkins-x-chartmuseum')
  }
}
