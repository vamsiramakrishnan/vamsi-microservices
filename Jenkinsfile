pipeline {
  agent {
    dockerfile {
      filename 'Dockerfile'
    }

  }
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
        sh 'npm install'
        sh 'CI=true DISPLAY=:99 npm test'
        sh 'export VERSION=$PREVIEW_VERSION && skaffold build -f skaffold.yaml'
        sh 'jx step validate --min-jx-version 1.2.36'
        sh "jx step post build --image \$DOCKER_REGISTRY/$ORG/$APP_NAME:$PREVIEW_VERSION"
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
        git 'https://github.com/vamsiramakrishnan/vamsi-microservices.git'
        sh 'jx step validate --min-jx-version 1.1.73'
        sh 'echo $(jx-release-version) > VERSION'
        dir(path: './charts/vamsi-microservices') {
          sh 'make tag'
        }

        sh 'npm install'
        sh 'CI=true DISPLAY=:99 npm test'
        sh 'export VERSION=`cat VERSION` && skaffold build -f skaffold.yaml'
        sh 'jx step validate --min-jx-version 1.2.36'
        sh "jx step post build --image \$DOCKER_REGISTRY/$ORG/$APP_NAME:\$(cat VERSION)"
      }
    }
    stage('Promote to Environments') {
      when {
        branch 'master'
      }
      steps {
        dir(path: './charts/vamsi-microservices') {
          sh 'jx step changelog --version v$(cat ../../VERSION)'
          sh 'make release'
          sh 'jx promote -b --all-auto --timeout 1h --version $(cat ../../VERSION) --no-wait'
        }

      }
    }
  }
  environment {
    ORG = 'vamsiramakrishnan'
    APP_NAME = 'vamsi-microservices'
    CHARTMUSEUM_CREDS = credentials('jenkins-x-chartmuseum')
  }
}