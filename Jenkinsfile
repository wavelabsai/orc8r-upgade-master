pipeline {
    agent any
    options {
        buildDiscarder(logRotator(numToKeepStr: '3'))
    }
    environment {
        PUBLISH= "${WORKSPACE}/orc8r/tools/docker/publish.sh"
        REGISTRY= "registry.hub.docker.com/vik2595"
        MAGMA_TAG= "master"
    }
    stages {
        stage('Clone magma repo'){
            steps {
                git 'https://github.com/magma/magma.git'
            }
        }
        stage('Docker login'){
            steps {
                script{
                    sh 'echo 'f26e1a10-0106-4e6b-864d-4c44a058e235' | docker login --username vik2595 --password-stdin'
                }
            }
        }
        stage('Build orc8r images'){
            steps {
                script{
                    sh '${WORKSPACE}/orc8r/cloud/docker/build.py --all'                }
            }
        }
        stage('Publish orc8r images'){
            steps {
                script{
                    sh 'for image in controller nginx ; do ${env.PUBLISH} -r ${env.REGISTRY} -i ${image} -v ${env.MAGMA_TAG} ; done'                }
            }
        }
        stage('Build NMS images'){
            steps {
                script{
                    sh 'cd ${WORKSPACE}/nms/app/packages/magmalte && docker-compose build magmalte'                
                }
            }
        }
        stage('Publish NMS images'){
            steps {
                script{
                    sh 'COMPOSE_PROJECT_NAME=magmalte ${PUBLISH} -r ${REGISTRY} -i magmalte -v ${MAGMA_TAG}'                
                }
            }
        }
    }
}