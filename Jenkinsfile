pipeline {
    agent any
    options {
        buildDiscarder(logRotator(numToKeepStr: '3'))
    }
    environment {
        MAGMA_ROOT = "${WORKSPACE}"
        PUBLISH= "orc8r/tools/docker/publish.sh"
        DOCKER_ACCESS_TOKEN = credentials('VR_Docker_Token')
        REGISTRY = "vik2595"
        MAGMA_TAG = "master"
        GITHUB_REPO = "magma-helm-test"
        GITHUB_REPO_URL = "https://github.com/wavelabsai/magma-helm-test"
        GITHUB_USERNAME = "vik2595"
        GITHUB_ACCESS_TOKEN = credentials('VR_GH_PAT')
    }
    stages {
        stage('Clone magma repo'){
            steps {
                git 'https://github.com/magma/magma.git'
            }
        }
        stage('Docker login'){
            steps {
                    sh 'echo "f26e1a10-0106-4e6b-864d-4c44a058e235" | docker login --username vik2595 --password-stdin'
            }
        }

        stage('Build and publish orc8r images'){
            steps {
                dir ('orc8r/cloud/docker') {
                    sh """./build.py --all --nocache --parallel"""
                }
            }
        }

        stage('publish orc8r images'){
            steps {
                    sh """docker image tag orc8r_controller ${env.REGISTRY}/controller:${env.MAGMA_TAG}"""         
                    sh """docker push ${env.REGISTRY}/controller:${env.MAGMA_TAG}"""
                    sh """docker image tag orc8r_nginx ${env.REGISTRY}/nginx:${env.MAGMA_TAG}"""         
                    sh """docker push ${env.REGISTRY}/nginx:${env.MAGMA_TAG}"""
                    sh """docker rmi orc8r_controller orc8r_nginx"""
            }
        }

        stage('Build NMS images'){
            steps {
                dir ('nms/packages/magmalte') {
                    sh 'docker-compose build magmalte'
                }
            }
        }

        stage('Publish NMS images'){
            steps {
                    sh """docker image tag magmalte_magmalte ${env.REGISTRY}/magmalte:${env.MAGMA_TAG}"""         
                    sh """docker push ${env.REGISTRY}/magmalte:${env.MAGMA_TAG}"""
                    sh """docker rmi magmalte_magmalte"""
            }
        }
        stage('Publish Helm charts'){
            steps {
                    sh '''#!/bin/bash
                    mkdir -p ~/magma-charts && cd ~/magma-charts
                    git init
                    helm dependency update "${MAGMA_ROOT}/orc8r/cloud/helm/orc8r/"
                    helm package "${MAGMA_ROOT}/orc8r/cloud/helm/orc8r/" && helm repo index .
                    helm dependency update "${MAGMA_ROOT}/cwf/cloud/helm/cwf-orc8r/"
                    helm package "${MAGMA_ROOT}/cwf/cloud/helm/cwf-orc8r/" && helm repo index .
                    helm dependency update "${MAGMA_ROOT}/lte/cloud/helm/lte-orc8r/"
                    helm package "${MAGMA_ROOT}/lte/cloud/helm/lte-orc8r/" && helm repo index .
                    helm dependency update "${MAGMA_ROOT}/feg/cloud/helm/feg-orc8r/"
                    helm package "${MAGMA_ROOT}/feg/cloud/helm/feg-orc8r/" && helm repo index .
                    helm dependency update "${MAGMA_ROOT}/fbinternal/cloud/helm/fbinternal-orc8r/"
                    helm package "${MAGMA_ROOT}/fbinternal/cloud/helm/fbinternal-orc8r/" && helm repo index .
                    helm dependency update "${MAGMA_ROOT}/wifi/cloud/helm/wifi-orc8r/"
                    helm package "${MAGMA_ROOT}/wifi/cloud/helm/wifi-orc8r/" && helm repo index .
                    git add . && git commit -m "orc8r charts commit for version 1.5"
                    git push https://${GITHUB_REPO}:${GITHUB_ACCESS_TOKEN}@github.com/${GITHUB_USERNAME}/${GITHUB_REPO}.git
                '''         
            }
        }
    }
}