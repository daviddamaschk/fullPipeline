#!/usr/bin/env groovy
@Library('jenkins-pipeline-lib')
import de.mpc.pipeline.PropertyLoader

//noinspection GroovyAssignabilityCheck
pipeline {
    agent any

    tools {
        maven 'M3'
    }

    stages {
        stage('Checkout and load Parameters') {
            steps {
                git url: '${gitProjectUrl}',
                        credentialsId: 'git-credentials'

                script {
                    propertyLoader = new PropertyLoader(this)
                    propertyLoader.init()
                }

                sh 'mvn clean install'
            }
        }

        stage('Integrationtest') {
            steps {
                sh 'docker rm javaee || true'
                sh 'docker rm spark || true'

                script {
                    def image = docker.build("daviddamaschk/simplejavaee")
                    image.withRun("-p 7777:8080 -p 9990:9990 -p 8787:8787 --name javaee --net commonnet") { c ->
                        docker.image("sparkapp:latest").withRun("--net commonnet --name spark") {
                            c2 -> sh 'mvn verify -DskipITs=false'
                        }
                    }
                }
            }
        }

        stage('Push to Docker repository') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-credentials') {
                        docker.image('daviddamaschk/simplejavaee').push('latest')
                    }
                }
            }
        }
        /** TODO for next part
        stage('Sonar') {
            when {
                expression {
                    return propertyLoader.isSonarEnabled()
                }
            }

            steps {
                sh 'mvn clean package sonar:sonar'
                timeout(time: 1, unit: 'HOURS') {
                    withSonarQubeEnv('My SonarQube Server') {
                        sh 'mvn sonar:sonar'
                    }

                    script {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }
    **/


    }

    post {
        always {
            deleteDir()
        }
        failure {
            step([$class                  : 'Mailer',
                  notifyEveryUnstableBuild: true,
                  recipients              : "${propertyLoader.getMailRecipients()}",
                  sendToIndividuals       : true])
        }
    }

}