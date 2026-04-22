pipeline {
agent any


tools {
    maven 'maven3'
    jdk 'jdk21'
}

environment {
    GIT_REPO_URL   = 'https://github.com/devopsawspratice/Hotstar-02-04-26-.git'
    GIT_BRANCH     = 'main'

    DOCKER_USER    = 'devopsawspratice'
    APP_NAME       = 'hotstar-app'
    IMAGE_TAG      = "${env.BUILD_NUMBER}"

    DOCKER_IMAGE   = "${DOCKER_USER}/${APP_NAME}:${IMAGE_TAG}"
    DOCKER_LATEST  = "${DOCKER_USER}/${APP_NAME}:latest"

    SONARQUBE_ENV  = 'sq'

    AWS_REGION     = 'us-east-1'
    EKS_CLUSTER    = 'saran'

    // ⚠️ For practice only
    NEXUS_USER     = 'admin'
    NEXUS_PASS     = '12345678'
    DOCKER_PASS    = 'N@LD19Aran#'
}

stages {

    stage('Checkout') {
        steps {
            git branch: "${GIT_BRANCH}", url: "${GIT_REPO_URL}"
        }
    }

    stage('Build (Maven)') {
        steps {
            sh "mvn clean package -DskipTests"
        }
    }

    stage('Unit Test') {
        steps {
            sh "mvn test"
        }
    }

    stage('SonarQube Analysis') {
        steps {
            withSonarQubeEnv("${SONARQUBE_ENV}") {
                sh "mvn sonar:sonar"
            }
        }
    }

    stage('Quality Gate') {
        steps {
            timeout(time: 5, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
            }
        }
    }

    stage('Upload Artifact to Nexus') {
        steps {
            sh """
            cat > settings.xml <<EOF


<settings>
  <servers>
    <server>
      <id>maven-releases</id>
      <username>${NEXUS_USER}</username>
      <password>${NEXUS_PASS}</password>
    </server>
  </servers>
</settings>
EOF


            mvn deploy -s settings.xml
            """
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
            sh """
            echo "${DOCKER_PASS}" | docker login -u ${DOCKER_USER} --password-stdin
            docker push ${DOCKER_IMAGE}
            docker push ${DOCKER_LATEST}
            docker logout
            """
        }
    }

    stage('Deploy to EKS') {
        steps {
            sh """
            aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER}

            cp deployment.yml /tmp/deploy.yml
            sed -i "s|IMAGE_PLACEHOLDER|${DOCKER_IMAGE}|g" /tmp/deploy.yml

            kubectl apply -f /tmp/deploy.yml
            kubectl apply -f service.yml

            kubectl rollout restart deployment hotstardeploy
            kubectl rollout status deployment hotstardeploy
            """
        }
    }
}

post {
    success {
        echo "Pipeline SUCCESS 🚀"
    }
    failure {
        echo "Pipeline FAILED ❌"
    }
}


}
