pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        DOCKERHUB_USER = 'manesaurabh1704devops'
        AWS_REGION = 'ap-south-1'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        K8S_NAMESPACE = 'studentsphere'
        JAVA_HOME = '/usr/lib/jvm/java-17-openjdk-amd64'
        PATH = "/usr/lib/jvm/java-17-openjdk-amd64/bin:${env.PATH}"
    }

    stages {

        stage('Git Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Backend Maven Build') {
            steps {
                echo 'Building Backend JAR...'
                dir('backend') {
                    sh 'export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64 && java -version && mvn clean package -DskipTests'
                }
            }
        }

        stage('Frontend npm Build') {
            steps {
                echo 'Building Frontend...'
                dir('frontend') {
                    sh 'npm ci'
                    sh 'npm run build'
                }
            }
        }

        stage('Trivy Security Scan') {
            steps {
                echo 'Running Trivy security scan...'
                sh 'trivy fs --severity HIGH,CRITICAL --exit-code 0 ./'
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker images...'
                sh "docker build -t ${DOCKERHUB_USER}/studentsphere-backend:${IMAGE_TAG} -f backend/dockerfile backend/"
                sh "docker build -t ${DOCKERHUB_USER}/studentsphere-frontend:${IMAGE_TAG} -f frontend/dockerfile frontend/"
            }
        }

        stage('Docker Push to DockerHub') {
            steps {
                echo 'Pushing images to DockerHub...'
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh "docker push ${DOCKERHUB_USER}/studentsphere-backend:${IMAGE_TAG}"
                sh "docker push ${DOCKERHUB_USER}/studentsphere-frontend:${IMAGE_TAG}"
            }
        }

        stage('Deploy to EKS') {
            steps {
                echo 'Deploying to AWS EKS...'
                sh """
                    kubectl set image deployment/backend \
                      backend=${DOCKERHUB_USER}/studentsphere-backend:${IMAGE_TAG} \
                      -n ${K8S_NAMESPACE}

                    kubectl set image deployment/frontend \
                      frontend=${DOCKERHUB_USER}/studentsphere-frontend:${IMAGE_TAG} \
                      -n ${K8S_NAMESPACE}

                    kubectl rollout status deployment/backend -n ${K8S_NAMESPACE}
                    kubectl rollout status deployment/frontend -n ${K8S_NAMESPACE}
                """
            }
        }
    }

    post {
        always {
            echo 'Cleaning workspace...'
            cleanWs()
        }
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed! Check logs above.'
        }
    }
}
