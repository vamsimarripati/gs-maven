pipeline {
    agent any

    environment {
        SONAR_HOST_URL = 'http://13.233.93.12:9000' // SonarQube server URL over HTTP
        SONAR_PROJECT_KEY = 'org.springframework:gs-maven'
        SONAR_PROJECT_NAME = 'gs-maven'
        
        // Update Nexus URL to HTTP
        NEXUS_URL = 'http://13.233.245.91:8081/repository/maven-releases/'
        NEXUS_USERNAME = credentials('nexus-username')  // Adjust to your Jenkins credential ID
        NEXUS_PASSWORD = credentials('nexus-password')  // Adjust to your Jenkins credential ID
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
                    dir('complete') {
                        sh 'mvn clean package'
                    }
                }
            }
        }

        stage('Upload to Nexus') {
            steps {
                script {
                    echo "Uploading artifact to Nexus: complete/target/gs-maven-0.1.0.jar"
                    withCredentials([usernamePassword(credentialsId: 'nexus-credentials', passwordVariable: 'NEXUS_PASSWORD', usernameVariable: 'NEXUS_USERNAME')]) {
                        sh """
                            mvn deploy:deploy-file \
                            -Dfile=complete/target/gs-maven-0.1.0.jar \
                            -DrepositoryId=nexus \
                            -Durl=${NEXUS_URL} \
                            -Dusername=${NEXUS_USERNAME} \
                            -Dpassword=${NEXUS_PASSWORD}
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
