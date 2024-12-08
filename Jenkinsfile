pipeline {
    agent any

    tools {
        maven 'maven'
    }

    environment {
        SONAR_HOST_URL = 'http://13.233.93.12:9000' // SonarQube server URL
        SONAR_PROJECT_KEY = 'org.springframework:gs-maven'
        SONAR_PROJECT_NAME = 'gs-maven'
        TOMCAT_HOST = '65.0.168.203:8080'  // Only hostname and port
        TOMCAT_USER = 'admin'
        TOMCAT_PASSWORD = 'Sushmi@2001'
        TOMCAT_DEPLOY_URL = "http://${TOMCAT_USER}:${TOMCAT_PASSWORD}@${TOMCAT_HOST}/manager/text/deploy?path=/gs-maven&update=true"
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

                        // Debugging step to verify the presence of the JAR and WAR files
                        sh 'ls -l target/*.war target/*.jar'

                        // Archive the artifacts for reference
                        archiveArtifacts artifacts: '**/target/*.war, **/target/*.jar', allowEmptyArchive: true
                    }
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    def deployFile = ''
                    def warFile = findFiles(glob: '**/target/*.war')[0]?.path
                    def jarFile = findFiles(glob: '**/target/*.jar')[0]?.path

                    // Deploy WAR file to Tomcat if found
                    if (warFile) {
                        deployFile = warFile
                        echo "Deploying WAR file: ${deployFile} to Tomcat"
                        sh """
                            curl -u ${TOMCAT_USER}:${TOMCAT_PASSWORD} --upload-file ${deployFile} ${TOMCAT_DEPLOY_URL}
                        """
                    } else if (jarFile) {
                        deployFile = jarFile
                        echo "Found JAR file: ${deployFile}. JAR files cannot be deployed directly to Tomcat."
                    } else {
                        echo "No WAR or JAR file found for deployment!"
                        error "Neither a WAR nor a JAR file was found to deploy!"
                    }
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
