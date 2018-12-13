// Copyright 2018 Docker, Inc. All Rights Reserved

properties([buildDiscarder(logRotator(numToKeepStr: '20'))])

pipeline {
    agent none

    options {
        checkoutToSubdirectory('src/github.com/docker/compose-on-kubernetes')
    }

    stages {
        stage('Build images') {
            agent {
                label 'team-local && linux'
            }
            environment {
                IMAGE_PREFIX = 'kube-compose-'
                COMMIT_SHA1 = "$GIT_COMMIT"
            }
            steps {
                script {
                    env.TAG = env.TAG_NAME == null ? env.COMMIT_SHA1 : env.TAG_NAME
                    env.IMAGE_REPOSITORY = env.TAG_NAME ? 'docker' : 'dockereng'
                    env.IMAGE_REPO_PREFIX = env.IMAGE_REPOSITORY + '/' + env.IMAGE_PREFIX
                }
                sh 'env | sort'
                dir('src/github.com/docker/compose-on-kubernetes') {
                    ansiColor('xterm') {
                        sh "make -f docker.Makefile images"
                        sh "mkdir images"
                        sh '''
                        for image in controller controller-coverage api-server api-server-coverage installer e2e-tests; do
                            docker save $IMAGE_REPO_PREFIX$image:$TAG -o images/$IMAGE_PREFIX$image.tar
                        done
                        '''
                    }
                    stash name: 'images', includes: "images/*.tar"
                }
            }
            post {
                cleanup {
                    deleteDir()
                }
            }
        }
        stage('Tests') {
            parallel {
                stage('Test e2e kube 1.11') {
                    agent {
                        label 'team-local && linux'
                    }
                    environment {
                        IMAGE_PREFIX = 'kube-compose-'
                        COMMIT_SHA1 = "$GIT_COMMIT"
                        CLUSTER_NAME = "$GIT_COMMIT"
                        KUBE_VERSION = '1.11.5'
                    }
                    steps {
                        script {
                            env.TAG = env.TAG_NAME == null ? env.COMMIT_SHA1 : env.TAG_NAME
                            env.IMAGE_REPOSITORY = env.TAG_NAME ? 'docker' : 'dockereng'
                            env.IMAGE_REPO_PREFIX = env.IMAGE_REPOSITORY + '/' + env.IMAGE_PREFIX
                        }
                        dir('src/github.com/docker/compose-on-kubernetes') {
                            unstash 'images'
                            sh './scripts/e2e-cluster.sh create'
                        }
                    }
                    post {
                        cleanup {
                            deleteDir()
                        }
                    }
                }
            }
        }
    }
}
