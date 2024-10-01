pipeline {
    agent any
    
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
        IMAGE_NAME = "kokareamol/bankapp"
        TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('mvn compile') {
            steps {
                sh "mvn compile "
            }
        }
        
        stage('trivy fs scan') {
            steps {
                sh "trivy fs --format table -o fs.txt ."
            }
        }
        
        stage('sonar scanning') {
            steps {
                withSonarQubeEnv('sonar-server') {
                   sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=multitier -Dsonar.projectName=multitier -Dsonar.java.binaries=target"
               }
            }
        }
        
        stage('quality gates') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: false
               }
            }
        }
        
        stage('mvn package/build') {
            steps {
                sh "mvn package -DskipTests=true "
            }
        }
        
        stage('mvn deploy/publish nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-settings', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                   sh "mvn deploy -DskipTests=true "
                }
            }
        }
        
        stage('docker build and tag') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-token') {
                        sh "docker build -t ${IMAGE_NAME}:${TAG} ."
                    }
                  }
                }
            }
        }
        
    }
}
