#!/usr/bin/env groovy

pipeline {
    agent any
    environment {
        NEW_VERSION = '1.3'
    }

    parameters {
        choice(name: 'VERSION', choices:['1','2', '3'], description: '')
        booleanParam(name: 'executeTest', defaultValue : true, description: '')
    }

    tools{
        maven 'maven-3.9.0'
    }

    stages {
        stage('init'){
            steps{
                script{
                    gv = load "script.groovy"
                }
            }
        }
        stage('config'){
            steps{
                script{
                    gv.config()
                }
            }
        }
        stage('build') {
            steps {
                script{
                    echo 'building the application ...  '
                    echo "Software version is ${NEW_VERSION}"
                    sh 'mvn build-helper:parse-version versions:set -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.nextMinorVersion}.\\\${parsedVersion.incrementalVersion}\\\${parsedVersion.qualifier?}'
                    sh 'mvn clean package'
                    def version = (readFile('pom.xml') =~ '<version>(.+)</version>')[0][2]
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                    sh "docker build -t d21it185rajbapodra/external_devops:${IMAGE_NAME} ."
                    }
            }
        }
      stage('test') {
          when{
             expression{
                 params.executeTest
             }
          }
          steps {
                script{echo 'testing the application'
                sh 'mvn test'}
          }
        }
      stage('deploy') {
        input{
            message "Select the environment to deploy"
            ok "done"
            parameters{
                choice(name: 'Type', choices:['Dev','Test','Deploy'], description: '')
            }

        }
            steps {
                script{echo 'deploying the application'
                withCredentials([usernamePassword(credentialsId: 'b56a95bf-6857-4f0c-a19e-75e890d83203', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]){
                    sh "echo ${PASSWORD} | docker login -u ${USERNAME} --password-stdin"
                    sh "docker push d21it185rajbapodra/external_devops:${IMAGE_NAME}"
                }}

             }
        }
    }
    post{
        always{
            echo 'Executing always...'
        }
        success{
            echo 'Executing success'
        }
        failure{
            echo 'Executing failure'
        }
    }
}