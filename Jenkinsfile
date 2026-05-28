pipeline {
    agent { label 'built-in' }

    environment {
        DOCKER = '/tmp/docker/docker'
        IMAGE = "art201090/my-app:build-${BUILD_NUMBER}"
        K8S_DEPLOY = 'k8s/deployment.yaml'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/Stalker1301981-alt/my-app']]
                )
            }
        }

        stage('Build & Push Docker') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        ${DOCKER} build -t ${IMAGE} .
                        ${DOCKER} tag ${IMAGE} art201090/my-app:latest
                        echo ${DOCKER_PASS} | ${DOCKER} login -u ${DOCKER_USER} --password-stdin
                        ${DOCKER} push ${IMAGE}
                        ${DOCKER} push art201090/my-app:latest
                    """
                }
            }
        }

        stage('Update manifest') {
            steps {
                sh "sed -i 's|image: art201090/my-app:.*|image: ${IMAGE}|' ${K8S_DEPLOY}"
            }
        }

        stage('Commit & Push to Git') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-token', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                    sh """
                        git config user.name 'Jenkins CI'
                        git config user.email 'jenkins@my-app.local'
                        git add ${K8S_DEPLOY}
                        git commit -m 'Update image to build-${BUILD_NUMBER}'
                        git push https://\${GIT_USER}:\${GIT_PASS}@github.com/Stalker1301981-alt/my-app.git HEAD:main
                    """
                }
            }
        }
    }
}
