pipeline {
    agent any

    environment {
        dev= 'python-dev'
        qa= 'python-qa'
        staging='python-stage'
        Tags= '$BUILD_NUMBER'
        dockerHubRegistryID = 'sagarppatil27041992'
        versionTags= versiontags(Tags)
    }

    stages {
        stage('Dev-StaticCodeAnalysis') {
            when {
                branch 'main'
            }
            steps {
                   sh '''#!/bin/bash 
                   sudo docker rm -f flake8
                   CONTAINER_python=$(sudo docker run -d -t -e PYTHONUNBUFFERED=0 -w /root -v ${PWD}:/root  --name flake8 python:3.7-alpine /bin/sh)
                   sudo docker exec -i $CONTAINER_python /bin/sh  -c "pip install --upgrade pip"
                   sudo docker exec -i $CONTAINER_python /bin/sh  -c "pip install flake8 && flake8 --exit-zero --format=pylint app.py/ >flake8-out.txt"
                   sudo docker exec -i $CONTAINER_python /bin/sh  -c "ls -lrth && pwd"
                       
                       #python3.7 -m virtualenv my-venv 
                        #source  my-venv/bin/activate
                        #Pip install flake8
                        #flake8 --format=pylint  notification_service/
                        #deactivate
                   '''    
                }
              //publish flake8 report to Jenkins
                post {
                  always {
                      echo "recording issues"
                      recordIssues(tools: [flake8(pattern: 'flake8-out.txt')])
                      echo "recorded......!"
                  }
                }
        }

        stage('Dev-PublishToSonarQube') {
            when {
                branch 'main'
            }
            environment {
             scannerHome = tool 'sonarscanner4'
            }
            options { skipDefaultCheckout() }
            steps {
               withSonarQubeEnv('sonar-qube') {
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=python-hello-world \
                    -Dsonar.sources=. \
                    -Dsonar.python.flake8.reportPaths=flake8-out.txt \
                    # -Dsonar.python.xunit.reportPath=nosetests.xml \
                    # -Dsonar.python.coverage.reportPaths=coverage.xml 
                    # -Dsonar.tests=notificationservice/ConsumerEx/ \
                    #-Dsonar.python.xunit.skipDetails=false \
                    '''
                } 
                //echo "hello"
            
                // abort the job if QualityGate fails.
                /*
                    timeout(time: 10, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                 }  */
            }
        }
        stage ("Dev-docker build") {
            when {
                branch 'develop'   
            }
            options { skipDefaultCheckout() }
            steps{
                imageBuild(dockerHubRegistryID,dev,Tags) // calling image build function to build image for dev environment
            }
        }
        stage('Dev-Docker Publish') {
            when {
                branch 'develop'   
            }
            options { skipDefaultCheckout() }
            steps {
               withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
                   
                    // calling pushToImage function to push image for dev environment to docker hub registry
                     pushToImage(dockerHubRegistryID,dev,dockerHubUser,dockerHubPassword,Tags)
                
                   // remove the image once its pushed to dockerhub registry from local
                    deleteImages(dockerHubRegistryID,dev,Tags) 

                }
            }
        }

        stage('Dev-Deploy') {
            when {
                branch 'develop'   
            }
            options { skipDefaultCheckout() }
            steps {
               withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
                   
                  // we run the docker image  that we build in previous steps by pulling it from docker hub registry
                    deploy(dockerHubRegistryID,dev,dockerHubUser,dockerHubPassword,Tags)
                }
            }
        }
        stage('Push GitTag') {
            when {
                branch 'main'
            }
            steps {
                withCredentials([gitUsernamePassword(credentialsId: 'github-cred', gitToolName: 'Default')]) {
                   sh  "git tag $versionTags"
                   // here we tag master branch
                   sh "git push origin $versionTags"
                   // here push the git tag
                }
            }
        }
        stage ("BuildDockerImage") {
            when {
                branch 'main'   
            }
            steps{
                imageBuild(dockerHubRegistryID,qa,Tags) 
                // calling image build function for qa env
                
            }
        }
    }
}
  
  
// define function to build docker images
void imageBuild(registry,env,Tags) {
    
    sh "sudo docker build --rm -t $registry/$env:$Tags --pull --no-cache . "
    echo "Image build complete"
}


// define function to push images
void pushToImage(registry,env,dockerUser,dockerPassword,Tags) {
    
    sh "sudo docker login -u $dockerUser -p $dockerPassword " 
    sh "sudo docker push $registry/$env:$Tags"
    echo "Image Push $registry/$env:$Tags completed"
}

// function to delete image from local

void deleteImages(registry,env,Tags) {

    sh "sudo docker rmi $registry/$env:$Tags"
    echo "Images deleted"
}

// function to deploy a container to an environment by pulling the image from docker hub registry
void deploy(registry,env,dockerUser,dockerPassword,Tags){
    sh "sudo docker login -u $dockerUser -p $dockerPassword "
    sh "sudo docker run -d --name java-app-$env-$Tags -p 3001:8080 $registry/$env:$Tags "   
}


// function to deploy a container to an environment by pulling the image from docker hub registry
void versiontags(Tags) {
    def tag= "Release-V-$Tags-0.0"
   return tag
}