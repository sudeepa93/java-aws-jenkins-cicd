pipeline {
    agent any

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "M3"
        jdk "JDK 11"
    }
    
    environment {
        DOCKER_IMAGE_NAME = 'hello-world-app-image-name'  // Set your Docker image name
        DOCKER_TAG = '1.0.0'  // Optionally set a tag for your image
    }
    

    stages {
    	stage('Checkout') {
            steps {
                // Get some code from a GitHub repository
                git branch: 'main', url: 'https://github.com/sudeepa93/java-aws-jenkins-cicd'
            }
        }
    
        stage('Build') {
            steps {

                // Run Maven on a Unix agent.
                sh "mvn -Dmaven.test.failure.ignore=true clean package"

                // To run Maven on a Windows agent, use
                // bat "mvn -Dmaven.test.failure.ignore=true clean package"
            }

            post {
                // If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.
                success {
                    junit '**/target/surefire-reports/TEST-*.xml'
                    archiveArtifacts 'target/*.jar'
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                // Build the Docker image
                script {
                    def dockerImage = docker.build("${DOCKER_IMAGE_NAME}:${DOCKER_TAG}")
                }
            }
        }
        
        stage('Publish Docker Image') {
            steps {
                // Optionally, push the Docker image to a registry
                script {
                    docker.withRegistry('http://localhost:5000/v2', 'docker-registry-credentials') {
                        docker.image("${DOCKER_IMAGE_NAME}:${DOCKER_TAG}").push()
                    }
                }
            }
        }
    }
}
