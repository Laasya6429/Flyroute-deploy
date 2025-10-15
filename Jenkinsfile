pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker')
        DOCKER_USER = "${DOCKERHUB_CREDENTIALS_USR}"
        DOCKER_PASS = "${DOCKERHUB_CREDENTIALS_PSW}"
        FRONTEND_ENV = credentials('frontend-env')
    }
    stages {
        stage('Clean Workspace & Checkout') {
            steps {
                cleanWs()
                git branch: 'main', url: 'https://github.com/Laasya6429/Flyroute-deploy.git'
            }
        }
        stage('Prepare .env') {
            steps {
                withCredentials([
                    string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY'),
                    string(credentialsId: 'GEMINI_API_KEY', variable: 'GEMINI_API_KEY')
                ]) {
                    sh '''
                    cat > backend/.env <<EOL
                    PORT=5000
                    ADMIN_USERNAME=admin
                    ADMIN_PASSWORD=admin123
                    JWT_SECRET=PLAYSUPER
                    DATABASE_URL='postgresql://neondb_owner:npg_n5X4eWVJybqI@ep-wandering-waterfall-a112knmz-pooler.ap-southeast-1.aws.neon.tech/neondb?sslmode=require&channel_binding=require'
                    AWS_S3_REGION=ap-south-1
                    AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
                    AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
                    AWS_S3_BUCKET_NAME=airrf
                    GEMINI_API_KEY=${GEMINI_API_KEY}
                    EOL
                                        '''
                }
            }
        }
        stage('Prepare Frontend .env') {
            steps {
                writeFile file: 'frontend/.env', text: "${FRONTEND_ENV}"
            }
        }
        stage('Docker Login') {
            steps {
                sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
            }
        }
        stage('Remove Old Containers & Images') {
            steps {
                sh '''
                    docker rm -f airfare-backend || true
                    docker rm -f airfare-frontend || true
                    docker rm -f airfare-db || true
                    docker rmi -f $DOCKER_USER/flyroute-backend:latest || true
                    docker rmi -f $DOCKER_USER/flyroute-frontend:latest || true
                '''
            }
        }
        stage('Build with Docker Compose') {
            steps {
                sh 'docker-compose -f docker-compose.yaml build --no-cache'
            }
        }
        stage('Push Images to DockerHub') {
            steps {
                sh """
                docker tag flyroute-backend:latest $DOCKER_USER/flyroute-backend:latest
                docker push $DOCKER_USER/flyroute-backend:latest
                docker tag flyroute-frontend:latest $DOCKER_USER/flyroute-frontend:latest
                docker push $DOCKER_USER/flyroute-frontend:latest
                """
            }
        }
        stage('Deploy Containers') {
            steps {
                sh '''
                    docker-compose down -v || true
                    docker-compose up -d
                '''
            }
        }
    }
    post {
        always {
            sh 'docker logout'
        }
    }
}
