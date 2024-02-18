#! /usr/bin/env groovy
pipeline {
  agent {
    label 'maven'
  }

  stages {
    stage('Build') {
      steps {
        echo 'Building..'
        git branch: 'openshift-jenkins-pipeline', url: 'https://github.com/timlam007/JenkinsPipeline.git'

        sh 'mvn clean package'
      }
    }

    stage('Create Container Image') {
      steps {
        echo 'Create Container Image..'
        script {
          openshift.withCluster() {
            openshift.withProject("tinlam-dev") {
              def buildConfigExists = openshift.selector("bc", "HelloWolrdApp").exists()

              if (!buildConfigExists) {
                openshift.newBuild("--name=HelloWolrdApp", "--docker-image=registry.redhat.io/jboss-eap-7/eap74-openjdk8-openshift-rhel7", "--binary")
              }

              openshift.selector("bc", "HelloWolrdApp").startBuild("--from-file=target/simple-servlet-0.0.1-SNAPSHOT.war", "--follow")
            }
          }
        }
      }
    }

    stage('Deploy') {
      steps {
        echo 'Deploying....'
        script {
          openshift.withCluster() {
            openshift.withProject("tinlam-dev") {
              def deployment = openshift.selector("dc", "HelloWolrdApp")

              if (!deployment.exists()) {
                openshift.newApp('HelloWolrdApp', "--as-deployment-config").narrow('svc').expose()
              }

              timeout(time: 5, unit: 'MINUTES') {
                openshift.selector("dc", "HelloWolrdApp").related('pods').untilEach(1) {
                  return (it.object().status.phase == "Running")
                }
              }
            }
          }
        }
      }
    }
  }
}
