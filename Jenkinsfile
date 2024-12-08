pipeline {
    agent any

    tools {
        maven 'maven'
    }

    environment {
        SONAR_HOST_URL = 'http://13.233.93.12:9000' // SonarQube server URL
        SONAR_PROJECT_KEY = 'org.springframework:gs-maven'
        SONAR_PROJECT_NAME = 'gs-maven'
        NEXUS_URL = 'http://13.233.245.91:8081/repository/maven-releases/' // Nexus HTTP URL
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

        stage('Create settings.xml for Nexus') {
            steps {
                script {
                    // Create settings.xml for Nexus credentials
                    writeFile file: 'settings.xml', text: """
                    <settings xmlns="http://maven.apache.org/settings/1.0.0"
                             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                             xsi:schemaLocation="http://maven.apache.org/settings/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
                        <servers>
                            <server>
                                <id>nexus-releases</id>
                                <username>${env.NEXUS_USERNAME}</username>
                                <password>${env.NEXUS_PASSWORD}</password>
                            </server>
                        </servers>
                    </settings>
                    """
                }
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
                        // Clean and build the project
                        sh 'mvn clean package'
                    }
                }
            }
        }

        stage('Upload to Nexus') {
            steps {
                script {
                    // Use findFiles to locate the artifact (jar file)
                    def artifactFile = findFiles(glob: '**/target/*.jar')[0]?.path
                    if (artifactFile) {
                        echo "Uploading artifact ${artifactFile} to Nexus"
                        withCredentials([usernamePassword(credentialsId: 'nexus-credentials', usernameVariable: 'NEXUS_USERNAME', passwordVariable: 'NEXUS_PASSWORD')]) {
                            sh """
                                mvn deploy:deploy-file \
                                    -Dfile=${artifactFile} \
                                    -DrepositoryId=nexus-releases \
                                    -Durl=${NEXUS_URL} \
                                    -DgroupId=com.example \
                                    -DartifactId=gs-maven \
                                    -Dversion=0.1.0-SNAPSHOT \
                                    -Dpackaging=jar \
                                    -s settings.xml
                            """
                        }
                    } else {
                        error "No artifact found to upload to Nexus"
                    }
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    // Use findFiles to locate the WAR file
                    def warFile = findFiles(glob: '**/target/*.war')[0]?.path
                    if (warFile) {
                        echo "Deploying ${warFile} to Tomcat"
                        sh """
                            curl -u ${TOMCAT_USER}:${TOMCAT_PASSWORD} --upload-file ${warFile} ${TOMCAT_DEPLOY_URL}
                        """
                    } else {
                        error "No WAR file found to deploy to Tomcat"
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
