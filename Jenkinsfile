def dockerImage

pipeline {
    agent {
        kubernetes {
            label 'jenkins-agent-pod'
            idleMinutes 5
            yamlFile 'jenkins-agent-build-pod.yaml'
            defaultContainer 'ez-docker-helm-build'
        }
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }

    environment {
        DOCKER_IMAGE = 'irronroman19/task-app'
        DOCKER_CREDENTIALS_ID = 'docker-token'
        GITHUB_REPO = 'IrronRoman19/final-project-devops-sela'
    }

    stages {
        stage('Setup') {
            steps {
                checkout scm
                script {
                    // Define inline environment setup and utility functions
                    def initEnv = {
                        // Placeholder for environment setup logic
                        echo 'Environment setup initialized'
                    }

                    def getUniqueBuildIdentifier = {
                        // Generate a unique identifier, e.g., timestamp
                        return System.currentTimeMillis().toString()
                    }

                    // Initialize environment
                    initEnv()

                    def id = getUniqueBuildIdentifier()
                    if (env.BRANCH_NAME == 'main') {
                        env.BUILD_ID = "1." + id
                    } else {
                        env.BUILD_ID = "0." + getUniqueBuildIdentifier() + "." + id
                    }
                    currentBuild.displayName += " {build-name:" + env.BUILD_ID + "}"
                }
            }
        }

        stage('Clone Repository') {
            steps {
                git branch: env.BRANCH_NAME, url: "https://github.com/${env.GITHUB_REPO}.git"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE}:${env.BUILD_ID}")
                }
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh 'pytest app'
            }
        }

        stage('Build Helm Package') {
            steps {
                sh 'helm package ./helm/task-app'
            }
        }

        stage('Push Docker Image') {
            when {
                branch 'main'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', env.DOCKER_CREDENTIALS_ID) {
                        dockerImage.push("${env.BRANCH_NAME}-${env.BUILD_ID}")
                        dockerImage.push('latest')
                    }
                }
            }
        }

        stage('Push Helm Package') {
            when {
                branch 'main'
            }
            steps {
                script {
                    sh "helm push ./helm/task-app"
                }
            }
        }
    }
}
