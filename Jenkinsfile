pipeline {
    agent any

    environment {
        TOMCAT_URL = "http://65.0.168.203:8080/manager/text/deploy?path=/gs-maven"
        TOMCAT_USER = credentials('tomcat-username') // Jenkins stored credentials for username
        TOMCAT_PASS = credentials('tomcat-password') // Jenkins stored credentials for password
        TOMCAT_HOME = '/opt/tomcat'  // Adjust this to your Tomcat installation path
        SONARQUBE_SERVER = 'SonarQube'  // Jenkins SonarQube server name
    }

    stages {
        stage('Prepare Tomcat Configuration') {
            steps {
                script {
                    echo 'Configuring Tomcat with updated permissions...'
                    sh """
                        cp custom-tomcat-users.xml ${TOMCAT_HOME}/conf/tomcat-users.xml
                        ${TOMCAT_HOME}/bin/shutdown.sh
                        ${TOMCAT_HOME}/bin/startup.sh
                    """
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    echo 'Running SonarQube analysis...'

                    // Ensure SonarQube scanner is installed and configured in Jenkins
                    withSonarQubeEnv('SonarQube') {
                        // Run SonarQube analysis using Maven or Gradle
                        sh 'mvn clean verify sonar:sonar'
                    }
                }
            }
        }

        stage('Build Application') {
            steps {
                script {
                    echo 'Building application...'

                    // Run your build tool (e.g., Maven or Gradle)
                    sh 'mvn clean install'

                    // Ensure the JAR file exists
                    def jarFile = findFiles(glob: 'complete/target/*.jar')[0].path
                    echo "Build successful, found JAR file: ${jarFile}"
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    // Find the JAR file to deploy
                    def jarFile = findFiles(glob: 'complete/target/*.jar')[0].path
                    echo "Deploying ${jarFile} to Tomcat"
                    
                    // Use curl to deploy the JAR file to Tomcat
                    sh """
                        curl -u ${TOMCAT_USER}:${TOMCAT_PASS} --upload-file ${jarFile} ${TOMCAT_URL}
                    """
                }
            }
        }

        stage('Post-Deployment Actions') {
            steps {
                echo 'Pipeline completed successfully!'
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }

        failure {
            echo 'Pipeline failed. Please check the logs for more details.'
        }
    }
}
