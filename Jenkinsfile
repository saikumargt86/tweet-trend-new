def registry = 'https://jenkins80.jfrog.io/'
pipeline {
    agent {
        node{
            label 'maven-server'
        }
    }
     parameters {
        string defaultValue: '857600833350', description: 'Enter your AWS account ID', name: 'AWS_ACCOUNT_ID'
        string defaultValue: 'ap-south-1', description: 'Enter your AWS account ID', name: 'AWS_DEFAULT_REGION'
        string defaultValue: 'ecr-repository-delete', description: 'Enter your AWS account ID', name: 'IMAGE_REPO_NAME'
        string defaultValue: 'latest2', description: 'Enter your AWS account ID', name: 'IMAGE_TAG'
        }
    environment {
        PATH = "/opt/apache-maven-3.9.5/bin:$PATH"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
        FILENAME = "${IMAGE_REPO_NAME}${IMAGE_TAG}.txt"
        }
    stages {
        stage('build') {
            steps {
                echo "---------------build started-------------------"
                sh 'mvn clean deploy -Dmaven.test.skip=true'
                echo "---------------build ended-------------------"
            }
        }
        stage('test') {
            steps {
                echo "---------------unit test started-------------------"
                sh 'mvn surefire-report:report'
                echo "---------------unit test ended-------------------"
            }
        }
       stage('SonarQube analysis') {
    environment{
         scannerHome = tool 'SonarQubeScanner-Tool';
    }
    steps{
    withSonarQubeEnv('sonarqube-server') { // If you have configured more than one global server connection, you can specify its name
      sh "${scannerHome}/bin/sonar-scanner"
    }
    }
  }
  stage("Quality Gate"){
    steps{
        script{
  timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
    def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
    if (qg.status != 'OK') {
      error "Pipeline aborted due to quality gate failure: ${qg.status}"
    }
  }
  }
  }
  }
         stage("Jar Publish") {
        steps {
          catchError(buildResult: 'SUCCESS') {
            script {
                    echo '<--------------- Jar Publish Started --------------->'
                     def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"jfrog-token"
                     def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                     def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "jarstaging/(*)",
                              "target": "libs-release-local/{1}",
                              "flat": "false",
                              "props" : "${properties}",
                              "exclusions": [ "*.sha1", "*.md5"]
                            }
                         ]
                     }"""
                     def buildInfo = server.upload(uploadSpec)
                     buildInfo.env.collect()
                     server.publishBuildInfo(buildInfo)
                     echo '<--------------- Jar Publish Ended --------------->'  
            
            }
          }
        }   
    }

    stage(" Docker Build ") {
      steps {
        script {
           echo '<--------------- Docker Build Started --------------->'
            dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
           echo '<--------------- Docker Build Ends --------------->'
        }
      }
    }

      stage("Trivy image scanning"){
          steps{
            catchError(buildResult: 'SUCCESS') {
              script{
              echo '<--------------- Trivy image scanning Started --------------->'              
              sh "trivy image ${IMAGE_REPO_NAME}:${IMAGE_TAG} > ${filename}"
              echo '<--------------- Trivy image scanning Ends --------------->'
            }
            }
          }
        }

            stage (" Docker Publish "){
        steps {
            script {
               echo '<--------------- Docker Publish Started --------------->'  
                 sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"
                sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
                }    
               echo '<--------------- Docker Publish Ended --------------->'  
            }
        }

         stage ('Helm Deploy') {
          steps {
            script {
                sh "helm upgrade release01 --install k8sdep --namespace helm-deployment --set image.tag=${IMAGE_TAG}"
                }
            }
        }

      
     }
        post{
          success{
              emailext attachLog: true, body: 'test message', subject: 'test message', to: 't4ssietsai86@gmail.com'
          }
        }
          
        
    } 

    //stage("install with helm")  {
    //    steps{
    //        script{
    //            sh 'helm install ttrend ttrent-0.1.0.tgz'
    //        }
    //    }
    //}

    






