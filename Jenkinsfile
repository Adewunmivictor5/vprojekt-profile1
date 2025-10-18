pipeline {
    agent any

    environment {
        // Docker image name and tag
        DOCKER_IMAGE = "adewunmivictor5/vprojekt-profile1:${env.BUILD_ID}"

        // Jenkins credentials ID for Docker Hub
        DOCKERHUB_CREDENTIALS = "dockerhub-credentials"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "🔄 Checking out source code..."
                git branch: 'main', url: 'https://github.com/Adewunmivictor5/vprojekt-profile1.git'
            }
        }

        stage('Build & Test with Maven') {
            steps {
                echo "⚙️ Building and testing application..."
                sh 'mvn clean package -DskipTests=false'
            }
        }

        stage('Generate Reports') {
            steps {
                echo "📊 Generating reports..."
                sh 'mvn checkstyle:checkstyle'
                sh 'mvn jacoco:report'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "🐳 Building Docker image..."
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                echo "📤 Pushing image to Docker Hub..."
                withCredentials([
                    usernamePassword(
                        credentialsId: dockerhub_credentials,
                        usernameVariable: 'DOCKERHUB_USER',
                        passwordVariable: 'DOCKERHUB_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin
                        docker push ${DOCKER_IMAGE}
                        docker tag ${DOCKER_IMAGE} adewunmivictor5/vprojekt-profile1:latest
                        docker push adewunmivictor5/vprojekt-profile1:latest
                    '''
                }
            }
        }

        stage('Deploy Container Locally') {
            steps {
                echo "🚀 Deploying container on EC2..."
                sh '''
                    docker rm -f vpro-container || true
                    docker run -d -p 9000:8080 --name vpro-container ${DOCKER_IMAGE}
                '''
                echo "🌐 Application deployed successfully at: http://<your-ec2-public-ip>:9000"
            }
        }
    }

    post {
        failure {
            echo '❌ Build failed! Check Jenkins console logs.'
        }
        success {
            echo "✅ Build, push, and deployment completed successfully!"
        }
    }
}

