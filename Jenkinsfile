pipeline {
    environment {
    imagename = "Qr-Code_generator"
    registryCredential = 'swapnil257-dockerhub'
    dockerImage = ''
    jenkinsProject = 'Qr-Code_generator'
    yamlFIleName = 'iotbackend'
  }

  agent any

  // tools { nodejs "node" }

  stages {
    stage("Git Staging") {
        steps{
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'ubuntu/****', url: 'https://github.com/swapnil2596/qrcode.git']]])
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
            sh 'docker image tag $imagename:v1.$BUILD_ID swapnil257/$imagename:v1.$BUILD_ID'
            sh 'docker image tag $imagename:v1.$BUILD_ID swapnil257/$imagename:latest'
            sh 'sudo docker image push swapnil257/$imagename:v1.$BUILD_ID'
            sh 'sudo docker image push swapnil257/$imagename:latest'
            sh 'docker image rmi -f $imagename:v1.$BUILD_ID swapnil257/$imagename:v1.$BUILD_ID sutradhar/$imagename:latest'
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
            sh 'sudo docker pull swapnil257/$imagename:latest'
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
