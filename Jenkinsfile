#!/usr/bin.env groovy

pipeline {   
    agent any
    stages {
        stage("test") {
            steps {
                script {
                    echo "Testing the application..."

                }
            }
        }
        stage("build") {
            steps {
                script {
                    echo "Building the application..."
                }
            }
        }

        stage("deploy") {
            steps {
                script {
                    def dockerCmd = 'docker run -p 3080:3080 -d njogud/react-nodejs-example:v1.0'
                    sshagent(['ec2-server-key']) {
                       //sh "ssh -o StrictHostKeyChecking=no ec2-user@40.176.133.134 ${dockerCmd}"  
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@40.176.133.134 'docker rm -f my-app || true && ${dockerCmd}'"
                    }
                }
            }
        }               
    }
} 
