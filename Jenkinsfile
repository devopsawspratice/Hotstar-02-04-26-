
pipeline {
    agent any

    environment {
        SONARQUBE_ENV = 'sq'
        DOCKER_IMAGE = "rajeshtutta123/rajesh_hotstar-02-04-26"
        AWS_CREDS = credentials('aws-creds')
        AWS_DEFAULT_REGION = 'us-east-1'
        RECIPIENTS = 'rajeshtutta123@gmail.com'
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/rajeshtutta/Hotstar-02-04-26-.git'
            }
        }
        stage('BUILD') {
        steps {
            sh 'mvn clean package -DskipTests'
        }
    }
        stage('JENKINS TO NEXUS') {
        steps {
          withMaven(jdk: 'jdk21', maven: 'maven3', traceability: true) {
             sh 'mvn deploy'
}
        }
    }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 6, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }


        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:latest .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin
                    docker push $DOCKER_IMAGE:latest
                    docker logout
                    '''
                }
            }
        }

    }
}
