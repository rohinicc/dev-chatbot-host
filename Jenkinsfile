pipeline {

agent any

parameters {
    choice(
        name: 'ACTION',
        choices: ['Deploy', 'Remove'],
        description: 'Deploy or Remove AI Chatbot application'
    )
}

environment {
    APP_NAME = "ai-chatbot"
    IMAGE_NAME = "rohinicc/ai-chatbot"
    IMAGE_TAG = "latest"
    DOCKERHUB_CREDENTIALS = "dockerhub-creds"
    COMPOSE_FILE = "docker-compose.yml"
}

stages {

    stage('Checkout Code') {
        steps {
            echo "📥 Cloning repository..."
            checkout scm
        }
    }

    stage('Build Spring Boot') {
        when {
            expression { params.ACTION == 'Deploy' }
        }
        steps {
            echo "⚙️ Building Spring Boot application..."
            sh './mvnw clean package -DskipTests'
        }
    }

    stage('Build Docker Image') {
        when {
            expression { params.ACTION == 'Deploy' }
        }
        steps {
            echo "🐳 Building Docker image..."
            sh '''
            docker build -t $IMAGE_NAME:$IMAGE_TAG .
            '''
        }
    }

    stage('Login to DockerHub') {
        when {
            expression { params.ACTION == 'Deploy' }
        }
        steps {
            echo "🔐 Logging into DockerHub..."
            withCredentials([usernamePassword(
            
              credentialsId: 'dockerhub-credentials',
              usernameVariable: 'DOCKER_USER',
               passwordVariable: 'DOCKER_PASS'
            )]) {
                sh '''
                echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                '''
            }
        }
    }

    stage('Push Docker Image') {
        when {
            expression { params.ACTION == 'Deploy' }
        }
        steps {
            echo "📦 Pushing Docker image to DockerHub..."
            sh '''
            docker push $IMAGE_NAME:$IMAGE_TAG
            '''
        }
    }

    stage('Docker Compose Deploy') {
        when {
            expression { params.ACTION == 'Deploy' }
        }
        steps {
            echo "🚀 Deploying application using Docker Compose..."
            sh '''
            docker-compose -f $COMPOSE_FILE up -d --build
            '''
        }
    }

    stage('Docker Compose Remove') {
        when {
            expression { params.ACTION == 'Remove' }
        }
        steps {
            echo "🧹 Removing containers..."
            sh '''
            docker-compose -f $COMPOSE_FILE down
            docker system prune -af
            '''
        }
    }
}

post {
    success {
        echo "✅ AI Chatbot deployment completed successfully 🚀"
    }
    failure {
        echo "❌ Pipeline failed. Check Jenkins logs!"
    }
}

}