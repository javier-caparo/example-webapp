def builderImage
def productionImage
def ACCOUNT_REGISTRY_PREFIX
def GIT_COMMIT_HASH

pipeline {
    agent any
    stages {
        stage('Checkout Source Code and Logging Into Registry') {
            steps {
                echo 'Logging Into the Private ECR Registry'
                script {
                    GIT_COMMIT_HASH = "123456789"
                    ACCOUNT_REGISTRY_PREFIX = "156823553040.dkr.ecr.us-west-2.amazonaws.com"
                    sh """
                    \$(aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 156823553040.dkr.ecr.us-west-2.amazonaws.com)
                    """
                }
            }
        }

        stage('Make A Builder Image') {
            steps {
                echo 'Starting to build the project builder docker image'
                script {
                    builderImage = docker.build("${ACCOUNT_REGISTRY_PREFIX}/example-webapp-builder:${GIT_COMMIT_HASH}", "-f ./Dockerfile.builder .")
                    builderImage.push()
                    builderImage.push("${env.GIT_BRANCH}")
                    builderImage.inside('-v $WORKSPACE:/output -u root') {
                        sh """
                           cd /output
                           lein uberjar
                        """
                    }
                }
            }
        }

        stage('Unit Tests') {
            steps {
                echo 'running unit tests in the builder image.'
                script {
                    builderImage.inside('-v $WORKSPACE:/output -u root') {
                    sh """
                       cd /output
                       lein test
                    """
                    }
                }
            }
        }

        stage('Build Production Image') {
            steps {
                echo 'Starting to build docker image'
                script {
                    productionImage = docker.build("${ACCOUNT_REGISTRY_PREFIX}/example-webapp:${GIT_COMMIT_HASH}")
                    productionImage.push()
                    productionImage.push("${env.GIT_BRANCH}")
                }
            }
        }
    }
}
