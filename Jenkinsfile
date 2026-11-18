```groovy
pipeline {
    agent any

    environment {
        // Docker configuration
        DOCKER_CREDENTIALS_ID = 'docker-cred'
        DOCKER_IMAGE          = 'docker777john/test1'

        // SonarQube configuration
        SONAR_SERVER_NAME     = 'sonarqube-env'
        SONAR_SCANNER_TOOL    = 'sonar-scanner'

        // GitHub configuration
        GIT_CREDENTIALS_ID    = 'github-cred'
        MANIFEST_REPO         = 'https://github.com/Johnjose10/test1.git'
        MANIFEST_BRANCH       = 'main'
    }

    stages {

        stage('Checkout Source Code') {
            steps {
                checkout scm
                echo 'Source code checked out successfully'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool "${SONAR_SCANNER_TOOL}"

                    withSonarQubeEnv("${SONAR_SERVER_NAME}") {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=test1-key \
                            -Dsonar.projectName=test1-project-test \
                            -Dsonar.sources=.
                        """
                    }
                }
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: "${DOCKER_CREDENTIALS_ID}",
                            usernameVariable: 'DOCKER_USER',
                            passwordVariable: 'DOCKER_PASS'
                        )
                    ]) {
                        sh '''
                            echo "$DOCKER_PASS" | docker login \
                            -u "$DOCKER_USER" --password-stdin

                            docker build \
                            -t ${DOCKER_IMAGE}:${BUILD_NUMBER} \
                            -t ${DOCKER_IMAGE}:latest \
                            -f Dockerfile .

                            docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                            docker push ${DOCKER_IMAGE}:latest

                            docker logout
                        '''
                    }
                }
            }
        }

        stage('Update Manifest Repo') {
            steps {
                dir('manifest-workspace') {

                    git branch: "${MANIFEST_BRANCH}",
                        credentialsId: "${GIT_CREDENTIALS_ID}",
                        url: "${MANIFEST_REPO}"

                    withCredentials([
                        usernamePassword(
                            credentialsId: "${GIT_CREDENTIALS_ID}",
                            usernameVariable: 'GH_USER',
                            passwordVariable: 'GH_TOKEN'
                        )
                    ]) {
                        sh '''
                            git remote set-url origin \
                            https://${GH_USER}:${GH_TOKEN}@github.com/Johnjose10/test1.git

                            git pull --rebase origin main

                            # Update Docker image in deployment.yaml
                            sed -i -E \
                            "s|^([[:space:]]*image:[[:space:]]*).*|\\1${DOCKER_IMAGE}:${BUILD_NUMBER}|" \
                            manifests/deployment.yaml

                            git config user.name "Johnjose10"
                            git config user.email "jhon123jose@gmail.com"

                            git add manifests/deployment.yaml

                            if ! git diff --cached --quiet; then
                                git commit -m "Update test1 image to ${BUILD_NUMBER} [skip ci]"
                                git push origin main
                            else
                                echo "No manifest changes to commit"
                            fi
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully! Docker image pushed and manifest repository updated.'
        }

        failure {
            echo 'Pipeline failed. Please review the stage logs above.'
        }
    }
}
```