pipeline {
    agent none
    options {
        buildDiscarder(logRotator(daysToKeepStr: '30'))
    }
    triggers { cron(env.BRANCH_NAME ==~ /^master$/ ? 'H H(0-6) 1 * *' : '') }
    stages {
        stage('Matrix') {
            matrix {
                axes {
                    axis {
                        name 'PLATFORM'
                        values 'linux-amd64', 'linux-arm64'
                    }
                }
                environment {
                    TAG_SUFFIX = "-${TAG_NAME ?: GIT_COMMIT}-${PLATFORM}"
                }
                stages {
                    stage('Build, Test, Publish') {
                        agent { label "${PLATFORM}" }
                        stages {
                            stage('Build') {
                                steps {
                                    sh 'docker compose build'
                                }
                            }
                            stage('Publish') {
                                environment {
                                    DOCKER_REGISTRY_CREDS = credentials('docker-registry-credentials')
                                }
                                when {
                                    buildingTag()
                                }
                                steps {
                                    sh 'echo "$DOCKER_REGISTRY_CREDS_PSW" | docker login --username "$DOCKER_REGISTRY_CREDS_USR" --password-stdin quay.io'
                                    sh 'docker compose push'
                                }
                                post {
                                    always {
                                        sh 'docker logout quay.io'
                                    }
                                }
                            }
                        }
                        post {
                            always {
                                sh 'docker compose down -v --rmi all'
                                cleanWs()
                            }
                        }
                    }
                }
            }
        }
        stage('Publish main image') {
            agent { label "linux-amd64" }
            environment {
                DOCKER_REGISTRY_CREDS = credentials('docker-registry-credentials')
                TAG_SUFFIX = "-${TAG_NAME}"
            }
            when {
                buildingTag()
            }
            steps {
                sh './manifest-push.sh'
            }
            post {
                always {
                    sh 'docker logout quay.io'
                    cleanWs()
                }
            }
        }
    }
}
