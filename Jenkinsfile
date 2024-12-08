pipeline {
    agent any

    tools {
        maven 'maven'
    }

    environment {
        SONAR_HOST_URL = 'http://13.233.93.12:9000' // SonarQube server URL
        SONAR_PROJECT_KEY = 'org.springframework:gs-maven'
        SONAR_PROJECT_NAME = 'gs-maven'
        NEXUS_CREDENTIALS_ID = 'nexus-credentials'  // Jenkins credential ID for Nexus
        TOMCAT_HOST = 'http://65.0.168.203:8080'
        TOMCAT_USER = 'admin'
        TOMCAT_PASSWORD = 'Sushmi@2001'
        TOMCAT_DEPLOY_URL = "http://${TOMCAT_HOST}/manager/text/deploy?path=/gs-maven&update=true"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/Rama3058/gs-maven.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    dir('complete') {
                        // Using usernamePassword for SonarQube credentials
                        withCredentials([usernamePassword(credentialsId: 'sonar', usernameVariable: 'SONAR_USER', passwordVariable: 'SONAR_TOKEN')]) {
                            sh """
                                mvn clean verify sonar:sonar \
                                    -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                                    -Dsonar.projectName=${SONAR_PROJECT_NAME} \
                                    -Dsonar.host.url=${SONAR_HOST_URL} \
                                    -Dsonar.login=${SONAR_TOKEN}
                            """
                        }
                    }
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    dir('complete') {
                        sh 'mvn clean package'
                        // List files in the target directory to verify JAR file creation
                        sh 'ls -l target/'
                    }
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    // Ensure that the 'target/' directory exists before attempting to find the .jar file
                    def targetDirExists = fileExists 'complete/target'
                    if (!targetDirExists) {
                        error "Target directory does not exist. Please check the build step."
                    }

                    // Use find command to search for .jar file in target directory
                    def artifactFile = sh(script: 'find complete/target/ -name "*.jar" -print -quit', returnStdout: true).trim()
                    
                    if (!artifactFile) {
                        error("JAR file not found. Please check the build step.")
                    }
                    
                    echo "Deploying ${artifactFile} to Tomcat"
                    sh """
                        curl -u ${TOMCAT_USER}:${TOMCAT_PASSWORD} --upload-file ${artifactFile} ${TOMCAT_DEPLOY_URL}
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Please check the logs.'
        }
    }
}
