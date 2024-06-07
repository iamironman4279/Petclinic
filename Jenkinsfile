pipeline {
    agent any 
    
    tools {
        jdk 'jdk11'
        maven 'maven3'
        // Assuming SonarScanner is already installed and configured in Jenkins
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    
    stages {
        stage("Git Checkout") {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/jaiswaladi246/Petclinic.git'
            }
        }
        
        stage("Compile") {
            steps {
                sh "mvn clean compile"
            }
        }
        
        stage("Test Cases") {
            steps {
                sh "mvn test"
            }
        }
        
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=Petclinic \
                        -Dsonar.projectKey=Petclinic '''
                }
            }
        }
        
        stage("OWASP Dependency Check") {
            steps {
                dependencyCheckPublisher additionalArguments: '--scan ./ --format HTML'
            }
        }
        
        stage("Build") {
            steps {
                sh "mvn clean install"
            }
        }
        
        stage("Docker Build & Push") {
            steps {
                script {
                    withDockerRegistry(credentialsId: '58be877c-9294-410e-98ee-6a959d73b352', toolName: 'docker') {
                        sh "docker build -t adijaiswal/pet-clinic123:latest ."
                        sh "docker push adijaiswal/pet-clinic123:latest"
                    }
                }
            }
        }
        
        stage("TRIVY Vulnerability Scan") {
            steps {
                sh "trivy image adijaiswal/pet-clinic123:latest"
            }
        }
        
        stage("Deploy To Tomcat") {
            steps {
                sh "cp target/petclinic.war /opt/apache-tomcat-9.0.65/webapps/"
            }
        }
    }
}
