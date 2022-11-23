#!/usr/bin/env groovy
pipeline {
    environment {
        ENV = "dev"
//         GOOGLE_CHAT_URI = 'https://chat.googleapis.com/v1/spaces/AAAAgnqc2No/messages?key=AIzaSyDdI0hCZtE6vySjMm-WEfRq3CPzqKqqsHI&token=SN68oqEGcv42knnoH0Wvj4xdbMeqaos5fVA-IgmSQFY%3D'

        // The VERSION env variable is a dash-concatenated bundle of three strings:
        //   1. The project pom artifact version minus "-SNAPSHOT".
        //   2. The first 5 chars of the git commit hash (you can ID specific code from an image).
        //   3. The timestamp of the build as year/month/day/hour/minute (so every build has a new image ID).
//         VERSION = sh(returnStdout:true, script: 'mvn exec:exec -Dexec.executable=awk -Dexec.args="\'BEGIN {printf(\\"%s-\\", gensub(/-SNAPSHOT/,\\"\\", \\"g\\", \\"\\${project.version}\\"));system(\\"{ echo -n `git rev-parse --short=5 HEAD` && echo -n "-" && date +%Y%m%d%H%M; }\\")}\'" --non-recursive -q').trim()
    }
    // options { skipDefaultCheckout() }

    agent {
        kubernetes {
            yaml '''
            apiVersion: v1
            kind: Pod
            metadata:
              labels:
                jenkins: agent
            spec:
              containers:
              - name: node
                image: node:16.13.0
                command:
                - cat
                tty: true
              - name: git-envsubst
                image: nexus01.evizi.com:8123/git-envsubst
                command:
                - cat
                tty: true
              - name: docker
                image: docker:latest
                command:
                - cat
                tty: true
                volumeMounts:
                - mountPath: /var/run/docker.sock
                  name: docker-sock
              volumes:
              - name: docker-sock
                hostPath:
                  path: /var/run/docker.sock
            '''
        }
    }

    stages {
//         stage('Build Project') {
//             when {
//                 anyOf {branch 'evizi-qa';}
//             }
//             steps {
//                 container('node') {
//                     sh '''
//                             rm -f package-lock.json
//                             rm -rf node_modules
//                             npm cache clean --force
//                             npm install --force
//                             npm run build
//                         '''
//                 }
//             }
//         }

        stage('Docker login') {
            steps {
                container('docker') {
                    withCredentials([usernamePassword(credentialsId: 'nexus01-creds-hung', passwordVariable: 'password', usernameVariable: 'username')]) {
                        sh '''
                            docker login -u $username -p $password nexus01.evizi.com:8123
                        '''
                    }
                }
            }
        }

        stage('Build and push docker image') {
            steps {
                container('docker') {
                    sh '''
                            docker build --tag shopizer-shop-reactjs:${BUILD_NUMBER} .
                            docker tag shopizer-shop-reactjs:${BUILD_NUMBER} nexus01.evizi.com:8123/shopizer-shop-reactjs:${BUILD_NUMBER}
                            docker push nexus01.evizi.com:8123/shopizer-shop-reactjs:${BUILD_NUMBER}
                        '''
                }
            }
        }

        stage('Docker logout') {
            steps {
                container('docker') {
                    sh '''
                            docker logout
                        '''
                }
            }
        }
//         stage('Update deployment') {
//             when {
//                 anyOf {branch 'evizi-qa';}
//             }
//             steps {
//                 container('git-envsubst') {
//                     withCredentials([gitUsernamePassword(credentialsId: 'hung.bang_creds', gitToolName: 'git-tool')]) {
//                         sh '''
//                             git clone http://gitlab.evizi.com/railinc_network/umler-ui-internal.git
//                             cd umler-ui-internal
//                             rm -rf deployment.yaml
//                             envsubst '$BUILD_NUMBER' < deployment_tpl.yaml > deployment.yaml
//                             git add .
//                             git commit -m "Updating"
//                             git push origin master
//
//                             rm -rf umler-ui-internal
//                         '''
//                     }
//                 }
//             }
//         }
    }
    post {

        always {
            echo 'Deleting workspace...'
            // deleteDir()
            echo 'Workspace deleted.'
        }
    }
}
