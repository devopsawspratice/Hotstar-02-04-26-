pipeline {
    agent any

    environment {
        GIT_REPO_URL      = 'https://github.com/devopsawspratice/Hotstar-02-04-26-.git'
        GIT_BRANCH        = 'main'

        DOCKERHUB_USER    = 'devopsawspratice'
        APP_NAME          = 'hotstar-app'
        IMAGE_TAG         = "${env.BUILD_NUMBER}"

        DOCKER_IMAGE      = "${DOCKERHUB_USER}/${APP_NAME}:${IMAGE_TAG}"
        DOCKER_LATEST     = "${DOCKERHUB_USER}/${APP_NAME}:latest"

        SONARQUBE_ENV     = 'sq'
        AWS_REGION        = 'us-east-1'
        EKS_CLUSTER       = 'saran'

        RECIPIENTS        = 'devopsawspratice@gmail.com'
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Cloning ${GIT_REPO_URL}"
                git branch: "${GIT_BRANCH}", url: "${GIT_REPO_URL}"
            }
        }

        stage('Build (Maven)') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Unit Test') {
            steps {
                sh 'mvn test'
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
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Upload Artifact to Nexus') {
            steps {
                withMaven(jdk: 'jdk21', maven: 'maven3') {
                    sh 'mvn deploy'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${DOCKER_IMAGE} -t ${DOCKER_LATEST} .
                """
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-cred',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${DOCKER_IMAGE}
                        docker push ${DOCKER_LATEST}
                        docker logout
                    """
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh """
                    aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER}

                    cp deployment.yml /tmp/deploy.yml
                    sed -i 's|IMAGE_PLACEHOLDER|${DOCKER_IMAGE}|g' /tmp/deploy.yml

                    kubectl apply -f /tmp/deploy.yml
                    kubectl apply -f service.yml

                    kubectl rollout status deployment/hotstar -n default
                """
            }
        }
    }


}
