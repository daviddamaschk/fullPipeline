#!/usr/bin/env groovy
//noinspection GroovyAssignabilityCheck
pipeline {
    agent any

    tools {
        maven 'M3_5_4'
    }

    stages {

        stage('Build Project') {
            steps {
                sh "mvn -v"
                sh 'mvn clean install -DskipTests'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'mvn test'
            }
        }


        stage("Deploy") {
            when {
                branch 'production'
            }

            steps {
                sh 'deploy.sh ...'
            }
        }

    }

    post {
        always {
            deleteDir()
        }
        failure {
            step([$class                  : 'Mailer',
                  notifyEveryUnstableBuild: true,
                  recipients              : "david.damaschk@exxcellent.de",
                  sendToIndividuals       : true])
        }
    }

}