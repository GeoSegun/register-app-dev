pipeline {
    agent { label 'Jenkins-Agent' }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    environment {
        APP_NAME = "register-app-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "durolusegun6"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
    }
    stages {
        // This stage cleans up the workspace
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        /*
        This stage checks out the code from the Git repository.
        - It uses the 'main' branch.
        - Authentication is handled using the 'GitHub' credentials.
        */
        stage("Checkout from SCM") {
            steps {
                git branch: 'main', credentialsId: 'GitHub', url: 'https://github.com/GeoSegun/register-app-dev.git'
            }
        }

        // This stage builds the application using Maven
        stage("Build Application") {
            steps {
                sh "mvn clean package"
            }
        }

        // This stage tests the application using Maven
        stage("Test Application") {
            steps {
                sh "mvn test"
            }
        }

        // This stage performs SonarQube analysis
        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }

        // This stage introduces a delay to allow SonarQube results to be processed
        stage("Delay") {
            steps {
                sleep time: 15, unit: 'SECONDS'
            }
        }

        // This stage waits for the SonarQube Quality Gate result
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }
            }
        }

        // This stage builds and pushes the Docker image to DockerHub
        stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('', DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('', DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }
       stage("Trivy Scan") {
           steps {
               script {
	            sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image durolusegun6/register-app-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
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
    }
}
