pipeline {
    agent { label 'built-in' }

    environment {
        DOCKER = '/tmp/docker/docker'
        IMAGE = "art201090/my-app:build-${BUILD_NUMBER}"
        K8S_DEPLOY = 'helm/my-app/values.yaml'
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

        stage('Test') {
            steps {
                sh '''
                    /tmp/docker/docker rm -f test-app 2>/dev/null || true
                    /tmp/docker/docker run --rm -d -p 8080:8080 --name test-app -v $PWD:/app python:3.11-alpine python -m http.server 8080 -d /app/app
                    sleep 3
                    /tmp/docker/docker exec test-app python -c "import urllib.request; print(urllib.request.urlopen('http://localhost:8080/app/index.html').read().decode())" | grep -q "Version"
                    /tmp/docker/docker stop test-app
                '''
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
                sh "sed -i 's|tag:.*|tag: build-${BUILD_NUMBER}|' ${K8S_DEPLOY}"
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
