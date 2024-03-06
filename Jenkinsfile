pipeline{
    agent any
    tools {
        jdk 'Java21'
        maven 'Maven3'
    }
    environment {
        APP_NAME = "complete-prodcution-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "abdullahyalcin"
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
        DOCKERHUB_USER = "abdullahyalcin"
        PATH="/usr/local/bin:$PATH"

    }
    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
            steps {
                git branch: 'main', url: 'https://github.com/abdullahyalcin/CI-CD-with-Kubernetes-Jenkins.git'
            }
        }
        
        stage('Build App Docker Image') {
            steps {
                echo 'Building App Image'                
                sh 'docker build --force-rm -t "$IMAGE_NAME" -f ./Dockerfile .'
                sh 'docker image ls'
            }
       }

        stage('Push Image to Dockerhub Repo') {
            steps {
                echo 'Pushing App Image to DockerHub Repo'
                withCredentials([string(credentialsId: 'docker-token', variable: 'DOCKERHUB_TOKEN')]) {
                sh 'docker login -u $DOCKERHUB_USER -p $DOCKERHUB_TOKEN'
                sh 'docker push "$IMAGE_NAME"'
                
            }
          }
        }

        stage("Trivy Scan") {
            steps {
                script {
                    sh "docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ${DOCKER_USER}/${APP_NAME}:${IMAGE_TAG} --no-progress --scanners vuln --exit-code 0 --severity HIGH,CRITICAL --format table"
                }
            }
        }


        stage ('Cleanup Artifacts') {
            steps {
                script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }

        stage("Trigger CD Pipeline") {
            steps {
                script {
                def localJenkinsUrl = 'http://localhost:8080'
                def localJenkinsJobPath = 'job/gitops-complete-pipeline'
                def localJenkinsToken = 'gitops-token'
            
            sh "curl -v -k --user abdullahyalcin:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' '${localJenkinsUrl}/${localJenkinsJobPath}/buildWithParameters?token=${localJenkinsToken}'"
        }
    }
}          
    }

    post {
        failure {
            emailext body: '''${SCRIPT, template="groovy-html.template"}''', 
                    subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Failed", 
                    mimeType: 'text/html',to: "apoyalcn98@gmail.com"
            }
         success {
               emailext body: '''${SCRIPT, template="groovy-html.template"}''', 
                    subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Successful", 
                    mimeType: 'text/html',to: "apoyalcn98@gmail.com"
          }      
    }
}
