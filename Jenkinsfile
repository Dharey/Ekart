pipeline {
    agent any
    tools{
        jdk "Java_JDK"
        maven "Java_Maven"
    }
    environment {
        JAVA_HOME = '/usr/lib/jvm/java-17-openjdk-amd64'
        PATH = "${JAVA_HOME}/bin:${env.PATH}"
        SERVICE_NAME = "shopping-cart"
        ORGANIZATION_NAME = "deetechpro"
        DOCKERHUB_USERNAME = "Docker_Username_Var"
        REPOSITORY_TAG = "${DOCKERHUB_USERNAME}/${ORGANIZATION_NAME}-${SERVICE_NAME}:${BUILD_ID}"
        //SCANNER_HOME= tool 'sonar-scanner'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, credentialsId: 'GitHubCred', poll: false, url: 'https://github.com/Dharey/Ekart.git'
            }
        }

        stage('Compile') {
           steps {
               sh 'java -version'
               sh "mvn clean compile -DskipTests=true"
            }
        }
        
        stage('Build') {
           steps {
               sh "mvn clean package -DskipTests=true"
            }
        }
        
        stage('Docker Build & Push') {
           steps {
                   withDockerRegistry([credentialsId: 'DOCKERHUB_USERNAME', url: ""]) {
                       sh 'docker build -t ${REPOSITORY_TAG} .'
                       sh 'docker push ${REPOSITORY_TAG}'
                   }
               }   
            }
        
        stage("Install kubectl"){
            steps {
                sh """
                    curl -LO https://storage.googleapis.com/kubernetes-release/release/`
                    curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
                    chmod +x ./kubectl
                    ./kubectl version --client
                """
            }
        }
        stage('Approval') {
            steps {
                // CD - Approval Button with a timeout of 15 minutes.
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Do you want to approve the deployment?', ok: 'Yes'
                }
                echo "Deployment Approved"
            }
        } 
        
        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'Kubernetes']) {
                script {
                    // Assuming kubectl is installed locally in the Jenkins environment
                    sh '''
                        # Substitute environment variables in deploymentservice.yml and apply to Kubernetes
                        envsubst < ${WORKSPACE}/deploymentservice.yml | ./kubectl apply -f -
                    '''
                    }
                }
            }
        }
        
    }
}
