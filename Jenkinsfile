pipeline {
    agent {label 'Jenkins-Agent'}
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }

    
    stages{

        // This stage cleans up the workspace
        stage("Cleanup Workspace"){
                steps {
                cleanWs()
                }
        }

        /*
        This stage checks out the code from the Git repository.
        - It uses the 'main' branch.
        - Authentication is handled using the 'GitHub' credentials.
        */
        stage("Checkout from SCM"){
                steps {
                    git branch: 'main', credentialsId: 'GitHub', url: 'https://github.com/GeoSegun/register-app-dev.git'
                }
        }

        // This stage builds the application using Maven.        
        stage("Build Application"){
            steps {
                sh "mvn clean package"
            }

        // This stage test the application   
       stage("Test Application"){
           steps {
                 sh "mvn test"
           }
       }

       }
}