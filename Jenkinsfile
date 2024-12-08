pipeline {
    agent any

    tools {
        maven 'maven'
    }

    environment {
        SONAR_HOST_URL = 'http://your-sonarqube-server.com' // SonarQube server URL
        SONAR_PROJECT_KEY = 'your-project-key'
        SONAR_PROJECT_NAME = 'your-project-name'
        NEXUS_URL = 'https://nexus.yourcompany.com/repository/maven-releases/'
        NEXUS_CREDENTIALS_ID = 'nexus-credentials'  // Jenkins credential ID for Nexus
        TOMCAT_HOST = 'your-tomcat-server-ip'
        TOMCAT_USER = 'your-tomcat-username'
        TOMCAT_PASSWORD = 'your-tomcat-password'
        TOMCAT_DEPLOY_URL = "http://${TOMCAT_USER}:${TOMCAT_PASSWORD}@${TOMCAT_HOST}:8080/manager/text/deploy?path=/gs-maven&update=true"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/Rama3058/gs-maven.git'
            }
        }

        // stage('SonarQube Analysis') {
        //     steps {
        //         script {
        //             sh """
        //                 mvn clean verify sonar:sonar \
        //                     -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
        //                     -Dsonar.projectName=${SONAR_PROJECT_NAME} \
        //                     -Dsonar.host.url=${SONAR_HOST_URL}
        //             """
        //         }
        //     }
        // }

        stage('Build') {
            steps {
                dir('complete') {
                    sh 'mvn clean package'
                }
            }
        }

        stage('Upload to Nexus') {
            steps {
                script {
                    // Upload the artifact to Nexus
                    def artifactFile = findFiles(glob: '**/target/*.jar')[0].path
                    echo "Uploading artifact ${artifactFile} to Nexus"
                    withCredentials([usernamePassword(credentialsId: "${NEXUS_CREDENTIALS_ID}", usernameVariable: 'NEXUS_USERNAME', passwordVariable: 'NEXUS_PASSWORD')]) {
                        sh """
                            mvn deploy:deploy-file \
                                -Dfile=${artifactFile} \
                                -DrepositoryId=nexus \
                                -Durl=${NEXUS_URL} \
                                -DgroupId=com.example \
                                -DartifactId=gs-maven \
                                -Dversion=0.1.0-SNAPSHOT \
                                -Dpackaging=jar
                        """
                    }
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    def warFile = findFiles(glob: '**/target/*.war')[0].path
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
