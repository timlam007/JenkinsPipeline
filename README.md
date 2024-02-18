# Code Like The Wind
Simple examples to help you build great software quickly

#Instructions: 

1- Add the jenkins app ( go to Developer view >> Add >> search for Jenkins (Ephemeral) >> click create ) 
To verify the creation of jenkins ,  click the "Topology" tab to view the progress of the Jenkins server. Select the Jenkins deployment. When it has finished deploying, you will see one pod in the “running” state. Also , you can go to Network tab > route > check the created route for jenkins , if all was okay , you will see the login page for jenkins ) 

2- Create Jenkins Pipeline : 

Clone this github repo and cd switch to 

> git clone https://github.com/timlam007/JenkinsPipeline.git
> git checkout openshift-jenkins-pipeline 
> cd codelikethewind

Then open the project folder in VS code or any IDE  , navigate to Jenkins file and edit the stages as below : 

For Buile : 

  stages {
    stage('Build') {
      steps {
        echo 'Building..'
        git branch: 'openshift-jenkins-pipeline', url: 'https://github.com/timlam007/JenkinsPipeline.git'

        sh 'mvn clean package'
      }
    }

For Container creation : 

    stage('Create Container Image') {
      steps {
        echo 'Create Container Image..'
        script {
          openshift.withCluster() {
            openshift.withProject("tinlam-dev") {
              def buildConfigExists = openshift.selector("bc", "helloworldapp").exists()

              if (!buildConfigExists) {
                openshift.newBuild("--name=helloworldapp", "--docker-image=registry.redhat.io/jboss-eap-7/eap74-openjdk8-openshift-rhel7", "--binary")
              }

              openshift.selector("bc", "helloworldapp").startBuild("--from-file=target/simple-servlet-0.0.1-SNAPSHOT.war", "--follow")
            }
          }
        }
      }
    }
For deployment to openshift : 

   stage('Deploy') {
      steps {
        echo 'Deploying....'
        script {
          openshift.withCluster() {
            openshift.withProject("tinlam-dev") {
              def deployment = openshift.selector("dc", "helloworldapp")

              if (!deployment.exists()) {
                openshift.newApp('helloworldapp', "--as-deployment-config").narrow('svc').expose()
              }

              timeout(time: 5, unit: 'MINUTES') {
                openshift.selector("dc", "helloworldapp").related('pods').untilEach(1) {
                  return (it.object().status.phase == "Running")
                }
              }
            }
          }
        }
      }
    }
  }


After change the jenkinsfile content , run below git commands to commit and push to your repo: 

> git add .
> git commit -m “Update jenkins pipeline”
> git push

3- Create Build Configuration

Select "Builds" in the console and then click on "Create BuildConfig".

Change the name of your build config to `my-pipeline`. Next, update the git uri and ref to match your git repository. This is where it’ll look for your Jenkins pipeline file.

uri: https://github.com/timlam007/JenkinsPipeline.git
ref: openshift-jenkins-pipeline

Click “Create” to create a new build config.

Then, the most important part is to update the BuildConfig yml file to update the Strategy section as below : 

strategy:
    jenkinsPipelineStrategy:
      jenkinsfilePath: Jenkinsfile


Finally, we’ll add a webhook to the build config. Open the OpenShift command window.







