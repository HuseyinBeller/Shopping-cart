pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }
    

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/HuseyinBeller/Shopping-cart.git'
            }
        }
        
        
        
         stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        
        
         stage('OWASP FileSystem Scan') {
            steps {
                dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        
        
         stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Shopping-card -Dsonar.projectKey=Shopping-card \
                        -Dsonar.java.binaries=. '''
                }
            }
        }
        
        
        
         stage('Build Application & Push Artifact') {
            steps {
               withMaven(globalMavenSettingsConfig: '', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: 'maven-settings-default', traceability: true) {
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }
        
        
         stage('Docker Image Build') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t huseyinbeller/shopping-card:latest -f docker/Dockerfile ." 
                    }
                }
            }
        }
        
        
        
         stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o trivy-report.html huseyinbeller/shopping-card:latest"
            }
        }
        
        
      stage('Docker Image Push Repository') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push huseyinbeller/shopping-card:latest " 
                    }
                }
            }
        }
        
         stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-auth', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.27.81:6443') {
                        sh "kubectl apply -f deploymentservice.yml"
                        sh "kubectl get svc -n webapps"
                }
            }
        }
    }
}
