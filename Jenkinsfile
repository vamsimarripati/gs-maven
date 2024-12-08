pipeline {
    agent any

    tools {
        maven 'maven'
    }

    environment {
        SONAR_HOST_URL = 'http://13.233.93.12:9000' // SonarQube server URL
        SONAR_PROJECT_KEY = 'org.springframework:gs-maven'
        SONAR_PROJECT_NAME = 'gs-maven'
        TOMCAT_HOST = 'http://65.0.168.203:8080'
        TOMCAT_USER = 'admin'
        TOMCAT_PASSWORD = 'Sushmi@2001'
        TOMCAT_DEPLOY_URL = "http://${TOMCAT_USER}:${TOMCAT_PASSWORD}@${TOMCAT_HOST}:8080/manager/text/deploy?path=/gs-maven&update=true"
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
                    // Ensure you're in the correct directory
                    dir('complete') {
                        // Clean and package the project
                        sh 'mvn clean package'

                        // Debugging step to verify the presence of the WAR file
                        sh 'ls -l target/*.war'

                        // Archive the artifacts for reference
                        archiveArtifacts artifacts: '**/target/*.war', allowEmptyArchive: true
                    }
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    def warFile = 'target/gs-maven-0.1.0.war'  // Ensure the WAR file name is correct
                    echo "Deploying ${warFile} to Tomcat"
                    sh """
                        curl -u ${TOMCAT_USER}:${TOMCAT_PASSWORD} --upload-file ${warFile} ${TOMCAT_DEPLOY_URL}
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
