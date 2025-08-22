pipeline {
  agent any
  options {
    skipDefaultCheckout(true)
    timestamps()
  }
  parameters {
    booleanParam(name: 'DOCKER_BUILD', defaultValue: true, description: 'Build Docker images')
    booleanParam(name: 'DEPLOY', defaultValue: false, description: 'Run docker compose up -d after build')
  }
  environment {
    PROJECT_ROOT = pwd()
    IMAGE_TAG = "${env.BRANCH_NAME ?: 'local'}-${env.BUILD_NUMBER}"
  }
  stages {
    stage('Checkout') {
      steps {
        checkout scm
        sh 'git submodule update --init --recursive || true'
      }
    }

    stage('Backend - Build & Test') {
      steps {
        dir('task-manager-backend') {
          script {
            docker.image('maven:3.9-eclipse-temurin-21').inside("-v ${env.WORKSPACE}/.m2:/root/.m2") {
              sh 'mvn -B -ntp clean verify'
            }
          }
        }
      }
    }

    stage('Frontend - Build') {
      steps {
        dir('task-manager-frontend') {
          script {
            docker.image('node:20-alpine').inside {
              sh 'npm ci'
              sh 'npm run build --if-present'
            }
          }
        }
      }
    }

    stage('Docker Build') {
      when { expression { return params.DOCKER_BUILD } }
      steps {
        script {
          docker.image('docker:27-cli').inside("-v /var/run/docker.sock:/var/run/docker.sock -v ${env.WORKSPACE}:${env.WORKSPACE} -w ${env.WORKSPACE}") {
            sh 'docker build -t task-manager-backend:${IMAGE_TAG} task-manager-backend'
            // Optionally build a production frontend image later (nginx)
          }
        }
      }
    }

    stage('Deploy (docker compose)') {
      when { expression { return params.DEPLOY } }
      steps {
        script {
          docker.image('docker:27-cli').inside("-v /var/run/docker.sock:/var/run/docker.sock -v ${env.WORKSPACE}:${env.WORKSPACE} -w ${env.WORKSPACE}") {
            sh 'docker compose up -d --build'
          }
        }
      }
    }
  }
  post {
    always {
      archiveArtifacts artifacts: 'task-manager-backend/target/*.jar', allowEmptyArchive: true
    }
  }
}
