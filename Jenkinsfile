pipeline {
    agent any
    tools{
        jdk "jdk17"
        maven "maven"
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, credentialsId: '22e67b0a-623b-4b3c-83c8-e3765dd6f400', poll: false, url: 'https://github.com/Dharey/Ekart.git'
            }
        }
            
        stage('Compile') {
           steps {
               sh "mvn clean compile -DskipTests=true"
            }
        }
        
        stage('OWASP SCAN') {
           steps {
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'Dependency_Security_Check'
        
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('SonarQube') {
           steps {
               withSonarQubeEnv('sonar-server') {
                   sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Shopping-Cart \
                   -Dsonar.java.binaries=. \
                   -Dsonar.projectKey=Shopping-Cart '''
               }
            }
        }
        
        stage('Build') {
           steps {
               sh "mvn clean package -DskipTests=true"
            }
        }
        
        stage('Docker Build & Push') {
           steps {
               script {
                   withDockerRegistry(credentialsId: '709434db-7916-42e4-b13f-a6254ef242b0', toolName: 'docker') {
                       
                       sh "docker build -t shopping-cart -f docker/Dockerfile ."
                       sh "docker tag shopping-cart oluwaseyi12/shopping-cart:latest"
                       sh "docker push oluwaseyi12/shopping-cart:latest"
                   }
               }   
            }
        }
        
        // stage('Docker Deploy') {
        //    steps {
        //        script {
        //            withDockerRegistry(credentialsId: '709434db-7916-42e4-b13f-a6254ef242b0', toolName: 'docker') {
                       
        //                sh "docker run -d --name shopping-website -p 8070:8070 oluwaseyi12/shopping-cart:latest"
        //            }
        //        }   
        //     }
        // }
    }
}
