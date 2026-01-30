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
        DOCKERHUB_USERNAME = "oluwaseyi12"
        REPOSITORY_TAG = "${DOCKERHUB_USERNAME}/${ORGANIZATION_NAME}-${SERVICE_NAME}:${BUILD_ID}"
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, credentialsId: 'GitHubCred', poll: false, url: 'https://github.com/Dharey/Ekart.git'
            }
        }

        stage('Run Sonar Analysis') {
            environment {
                SONAR_TOKEN = credentials('SONARQUBE_TOKEN')
        }
            steps {
                sh '''
                mvn clean verify -U sonar:sonar \
                    -Dsonar.projectKey=ekart-app-1 \
                    -Dsonar.organization=ekart-app-1 \
                    -Dsonar.host.url=https://sonarqube.deetechpro.com \
                    -Dsonar.login=$SONAR_TOKEN
                '''
            }
        }

        stage('Run SCA Analysis using Snyk') {
            steps {
                withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
                sh """
                    snyk auth $SNYK_TOKEN
                    mvn snyk:test -fn
                """
                }
            }
        }
        stage('Deploy to Nexus') {
            steps {
                // 'my-nexus-id' is the ID you gave the credential in Jenkins
                withCredentials([usernamePassword(credentialsId: 'nexus-deploy-creds', 
                                    usernameVariable: 'NEXUS_USER', 
                                    passwordVariable: 'NEXUS_PASSWORD')]) {
                    sh "mvn clean compile -DskipTests=true"
                    sh "mvn clean package -DskipTests=true"
                    sh 'mvn deploy -s settings.xml'
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
                withKubeConfig([credentialsId: 'kubernetes']) {
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
