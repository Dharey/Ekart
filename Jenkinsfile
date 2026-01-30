pipeline {
    agent any

    tools {
        jdk "Java_JDK"
        maven "Java_Maven"
    }

    environment {
        JAVA_HOME = '/usr/lib/jvm/java-17-openjdk-amd64'
        PATH = "${JAVA_HOME}/bin:${env.PATH}"
        SERVICE_NAME = "shopping-cart"
        ORGANIZATION_NAME = "deetechpro"
        REPOSITORY_TAG = "${DOCKERHUB_USERNAME}/${ORGANIZATION_NAME}-${SERVICE_NAME}:${BUILD_ID}"
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Dharey/Ekart.git', credentialsId: 'GitHubCred'
            }
        }

        stage('Run Sonar Analysis') {
            environment {
                SONAR_TOKEN = credentials('SONARQUBE_TOKEN')
            }
            steps {
                sh '''
                mvn clean
                rm -rf ~/.m2/repository/org/jacoco
                mvn clean verify sonar:sonar \
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
                    sh 'snyk auth $SNYK_TOKEN && mvn snyk:test -fn'
                }
            }
        }

        stage('Deploy to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-deploy-creds',
                        usernameVariable: 'NEXUS_USER',
                        passwordVariable: 'NEXUS_PASSWORD')]) {
                    sh 'mvn clean package deploy -s settings.xml -DskipTests=true'
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                withDockerRegistry([credentialsId: 'DOCKERHUB_USERNAME', url: '']) {
                    sh 'docker build -t ${REPOSITORY_TAG} .'
                    sh 'docker push ${REPOSITORY_TAG}'
                }
            }
        }

        stage('Install kubectl') {
            steps {
                sh '''
                curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
                chmod +x ./kubectl
                ./kubectl version --client
                '''
            }
        }

        stage('Approval') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Approve deployment?', ok: 'Yes'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'kubernetes']) {
                    sh '''
                    envsubst < ${WORKSPACE}/deploymentservice.yml | ./kubectl apply -f -
                    '''
                }
            }
        }
    }
}