pipeline {
    environment {
    imagename = "iotbackend-uat"
    registryCredential = 'sutradhar-dockerhub'
    dockerImage = ''
    jenkinsProject = 'iotbackend-uat'
    yamlFIleName = 'iotbackend'
  }

  agent any

  // tools { nodejs "node" }

  stages {
    stage("Git Staging") {
        steps{
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'Admin-Gitlab', url: 'https://gitlab.sutradhar.tech/aditi.deshpande/iotbackend.git']]])
            }
    }
     // Create Iamge of the Project
    stage('Login to Docker') {
      steps{
        script {
          docker.withRegistry( '', registryCredential ) {
          }
        }
      }
    }
    /////////////////////////////////////////////////
    // Create the Image                            //
    // Tag the Image                               //
    // Push the Image to Docker Hub                //
    // Remove the Images from Local Instance       //
    /////////////////////////////////////////////////
    stage('Build the Image') {
        steps{
            sh 'sudo su - jenkins -s/bin/bash'
            sh 'cd /var/lib/jenkins/workspace/$jenkinsProject/'
            sh 'docker image build -f Dockerfile -t $imagename:v1.$BUILD_ID .'
            sh 'docker image tag $imagename:v1.$BUILD_ID sutradhar/$imagename:v1.$BUILD_ID'
            sh 'docker image tag $imagename:v1.$BUILD_ID sutradhar/$imagename:latest'
            sh 'sudo docker image push sutradhar/$imagename:v1.$BUILD_ID'
            sh 'sudo docker image push sutradhar/$imagename:latest'
            sh 'docker image rmi -f $imagename:v1.$BUILD_ID sutradhar/$imagename:v1.$BUILD_ID sutradhar/$imagename:latest'
        }
    }


    /////////////////////////////////////////
    // Pull the Image from Docker Hub      //
    // Kill the Previous Container         //
    // Star the new Container              //
    /////////////////////////////////////////

    stage('Pulling Image from Docker') {
        agent {
          label 'kubenode'
        }
        steps {
            sh 'sudo docker pull sutradhar/$imagename:latest'
            // Check if the Deployment exists
         script {
                        def deploymentExists = sh(script: "kubectl get deployment ${jenkinsProject}-deployment", returnStatus: true) == 0

                        // Check if the Service exists
                        def serviceExists = sh(script: "kubectl get service ${jenkinsProject}-service", returnStatus: true) == 0

                        // Delete Deployment if it exists
                        if (deploymentExists) {
                            sh "kubectl delete deployment ${jenkinsProject}-deployment"
                        }

                        // Delete Service if it exists
                        if (serviceExists) {
                            sh "kubectl delete service ${jenkinsProject}-service"
                        }
         }

                    // Create or apply the YAML file
                    sh "kubectl apply -f ${yamlFIleName}.yaml"
        }
    }


    


    stage("Configuring Nginx") {
      agent {
        label 'kubenode'
      }
      steps{
        sh 'sudo cp iotbackend.conf /etc/nginx/sites-enabled/'
        sh 'sudo nginx -t'
        sh 'sudo systemctl reload nginx'
      }
    }

  }

}