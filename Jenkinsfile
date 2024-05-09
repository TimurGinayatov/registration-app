pipeline {
    agent any
    
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }

    environment {
	    APP_NAME = "register-app-pipeline"
            RELEASE = "1.0.0"
            DOCKER_USER = "timurginayatov17"
            DOCKER_PASS = 'dockerhub'
            IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
            IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
	    JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
    }	    
	

    stages {
        stage("Cleanup Workspace"){
                steps {
                cleanWs()
                }
        }
        stage("Checkout from SCM"){
                steps {
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/TimurGinayatov/registration-app'
                }
        }
        stage("Build Application"){
            steps {
                sh "mvn clean package"
                }

        }
        stage("Test Application"){
            steps {
                  sh "mvn test"
            }
        }
	    
        stage("SonarQube Analysis"){
            steps {
	        script {
	             withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') { 
                     sh "mvn sonar:sonar"
		     }
	        }	
            }
        }
	    
	stage("Quality Gate"){
            steps {
                script {
                     waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }	
            }
		
        }
	stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
		
	}
	stage("Trivy Scan") {
            steps {
                script {
	             sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock --name trivy aquasec/trivy image timurginayatov17/register-app-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
                }
            }
        }

	stage ('Cleanup Artifacts') {
            steps {
                script {
                     sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                     sh "docker rmi ${IMAGE_NAME}:latest"
	             sh "docker rm trivy"
                }
            }
        }
	stage("Trigger CD Pipeline") {
            steps {
                script {
                     sh "curl -v -k --user clouduser:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' '34.139.15.24:8080/job/gitops-register-app/buildWithParameters?token=gitops-token'"
                }
            }
       }
	    
    }	    
}

