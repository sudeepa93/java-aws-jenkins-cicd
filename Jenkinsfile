pipeline {
    agent any

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "M3"
        jdk "JDK 11"
    }
    
    environment {
    	AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '585768181909'
        ECR_REPO = 'my-test-repo-1'
        IMAGE_TAG = 'latest'  // Use ${env.BUILD_NUMBER} for unique tags if preferred
    
        DOCKER_IMAGE_NAME = 'hello-world-app-image-name'  // Set your Docker image name
        DOCKER_TAG = "${env.BUILD_NUMBER}"  // Optionally set a tag for your image
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
        
        stage('Tag Docker Image') {
            steps {
                script {
                    // Tag the Docker image for ECR
                    sh "docker tag ${DOCKER_IMAGE_NAME}:${DOCKER_TAG} ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}"
                }
            }
        }

        stage('Login to AWS ECR') {
            steps {
                withCredentials([aws(credentialsId: 'aws-jenkins-credentials', region: "${AWS_REGION}")]) {
                    // Log in to ECR Private
                    sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
                
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                script {
                    // Push the image to ECR
                    sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}"
                }
            }
        }
        
        //stage('Publish Docker Image') {
        //    steps {
        //        // Optionally, push the Docker image to a registry
        //        script {
        //            docker.withRegistry('http://localhost:5000/v2', 'docker-registry-credentials') {
        //                docker.image("${DOCKER_IMAGE_NAME}:${DOCKER_TAG}").push()
        //            }
        //        }
        //    }
        //}
    }
}
