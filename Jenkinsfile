#!/usr/bin/env groovy

library identifier: 'jenkins-shared-library@master', retriever: modernSCM(
    [$class: 'GitSCMSource',
    remote: 'https://github.com/Nyati1/jenkins-shared-library.git', 
    credentialsId: 'jenkins-access_v1' 
    ]
)

pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        IMAGE_NAME = 'njogud/demo-app:jma2.0'
    }
    
    stages {
        stage('increment version') {
            steps {
                script {
                    echo 'incrementing app version...'
                    sh 'mvn build-helper:parse-version versions:set \
                        -DnewVersion=\\${parsedVersion.majorVersion}.\\${parsedVersion.minorVersion}.\\${parsedVersion.nextIncrementalVersion} \
                        versions:commit'
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "${version}-${BUILD_NUMBER}"
                }
            }
        }
        
        stage('build app') {
            steps {
                echo 'building application jar...'
                buildJar()
            }
        }
        
        stage('build image') {
            steps {
                script {
                    echo 'building the docker image...'
                    buildImage(env.IMAGE_NAME)
                    dockerLogin()
                    dockerPush(env.IMAGE_NAME)
                }
            }
        } 
        
        stage("deploy") {
            steps {
                script {
                    echo 'deploying docker image to EC2...'
                    // def dockerCmd = "docker run -p 3080:3080 -d ${IMAGE_NAME}"
                    // def dockerCmd = "docker run -p 8080:8080 --name my-app -d ${IMAGE_NAME}"
                    // def dockerComposeCmd = "docker-compose -f docker-compose.yaml up --detach"
                    
                    // using shell script to run docker commands
                    
                    def shellCmd = "bash ./server-cmds.sh ${env.IMAGE_NAME}" 
                    
                    sshagent(['ec2-server-key']) {
                        // sh "ssh -o StrictHostKeyChecking=no ec2-user@40.176.133.134 ${dockerCmd}" 
                        // sh "ssh -o StrictHostKeyChecking=no ec2-user@40.176.133.134 'docker rm -f my-app || true && ${dockerCmd}'"
                        sh "scp server-cmds.sh ec2-user@40.176.133.134:/home/ec2-user"
                        sh "scp docker-compose.yaml ec2-user@40.176.133.134:/home/ec2-user"
                        // sh "ssh -o StrictHostKeyChecking=no ec2-user@40.176.133.134 ${dockerComposeCmd}"
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@40.176.133.134 ${shellCmd}"
                        
                        /* def shellCmd = "bash ./server-cmds.sh ${IMAGE_NAME}"
                        def ec2Instance = "ec2-user@18.184.54.160"

                        sshagent(['ec2-server-key']) {
                            sh "scp server-cmds.sh ${ec2Instance}:/home/ec2-user"
                            sh "scp docker-compose.yaml ${ec2Instance}:/home/ec2-user"
                            sh "ssh -o StrictHostKeyChecking=no ${ec2Instance} ${shellCmd}"*/
                    }
                }
            }                
        }
        
        stage('commit version update'){
            steps {
                script {
                     withCredentials([usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]){
                        // sh "git remote set-url origin https://$USER:$PASS@github.com:Nyati1/aws-multibranch-pipeline-module9_8_v1.git"
                         sh "git remote set-url origin https://$USER:$PASS@github.com/Nyati1/aws-multibranch-pipeline-module9_8_v1.git"
                        sh 'git add .'
                        sh 'git commit -m "ci: version bump"'
                        sh 'git push origin HEAD:jenkins-jobs'
                    }
                }
            }
        }
    }
}
