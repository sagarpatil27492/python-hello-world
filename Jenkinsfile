pipeline {
    agent any

    environment {
        dev= 'develop'
        qa= 'main'
        staging='stage'
        Tags= '$BUILD_NUMBER'
        dockerHubRegistryID = 'sagarppatil27041992'
        versionTags= versiontags(Tags)
    }

    stages {
        stage('Dev-StaticCodeAnalysis') {
            when {
                branch 'develop'
            }
            steps {
                   sh '''#!/bin/bash 
                   docker rm -f flake8
                   CONTAINER_python=$(docker run -d -t -e PYTHONUNBUFFERED=0 -w /root -v ${PWD}:/root  --name flake8 python:3.7-alpine /bin/sh)
                   docker exec -i $CONTAINER_python /bin/sh  -c "pip install flake8 && flake8 --exit-zero --format=pylint notification_service/ >flake8-out.txt"
                   docker exec -i $CONTAINER_python /bin/sh  -c "ls -lrth && pwd"
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
                  recordIssues(tools: [flake8(pattern: 'flake8-out.txt')])
                  }
                }
        }

        stage('Dev-PublishToSonarQube') {
            when {
                branch 'develop'
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
           
           //echo "hello"
            } 
                // abort the job if QualityGate fails.
            /*
                timeout(time: 10, unit: 'MINUTES') {
                     waitForQualityGate abortPipeline: true
                }  
            } */
 
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
