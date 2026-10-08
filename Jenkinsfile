pipeline {

    agent any

    environment {

        PROJECT_DIR = "/home/ubuntu/health_insurance_analytics"

        EC2_PUBLIC_IP = "16.113.32.243"

        MYSQL_ROOT_PASSWORD = "health123"

        DB_USER = "health"

        DB_PASSWORD = "health123"

        DB_NAME = "health_db"

        BACKEND_PORT = "8000"

        FRONTEND_PORT = "80"
    }

    stages {

        stage('Checkout') {
            steps {

                echo "======================================"
                echo "CHECKOUT SOURCE CODE"
                echo "======================================"

                checkout scm
            }
        }


        stage('Copy Project') {
            steps {

                echo "======================================"
                echo "COPY PROJECT TO DEPLOYMENT DIRECTORY"
                echo "======================================"

                sh '''
                    mkdir -p ${PROJECT_DIR}

                    cp -r ${WORKSPACE}/. ${PROJECT_DIR}/ 
                '''
            }
        }


        stage('Create Environment File') {
            steps {

                echo "======================================"
                echo "CREATING ENVIRONMENT FILE"
                echo "======================================"

                sh '''
                    cat > ${PROJECT_DIR}/.env <<EOF
MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}

DB_USER=${DB_USER}
DB_PASSWORD=${DB_PASSWORD}
DB_HOST=db
DB_PORT=3306
DB_NAME=${DB_NAME}

VITE_API_URL=http://${EC2_PUBLIC_IP}:${BACKEND_PORT}
CORS_ORIGINS=http://${EC2_PUBLIC_IP}
EOF

                    chmod 600 ${PROJECT_DIR}/.env
                '''
            }
        }


        stage('Validate Docker Compose') {
            steps {

                echo "======================================"
                echo "VALIDATING DOCKER COMPOSE"
                echo "======================================"

                sh '''
                    cd ${PROJECT_DIR}

                    docker compose config
		    docker compose down
                '''
            }
        }


        stage('Build Docker Images') {
            steps {

                echo "======================================"
                echo "BUILDING DOCKER IMAGES"
                echo "======================================"

                sh '''
                    cd ${PROJECT_DIR}

                    docker compose build --no-cache
                '''
            }
        }


        stage('Deploy Application') {
            steps {

                echo "======================================"
                echo "DEPLOYING APPLICATION"
                echo "======================================"

                sh '''
                    cd ${PROJECT_DIR}

                    docker compose up -d
                '''
            }
        }


        stage('Check Containers') {
            steps {

                echo "======================================"
                echo "CHECKING CONTAINERS"
                echo "======================================"

                sh '''
                    cd ${PROJECT_DIR}

                    docker compose ps
                '''
            }
        }


        stage('Backend Health Check') {
            steps {

                echo "======================================"
                echo "BACKEND HEALTH CHECK"
                echo "======================================"

                sh '''
                    echo "Testing backend..."

                    curl -f http://localhost:${BACKEND_PORT}/

                    echo ""

                    echo "Backend is working successfully"
                '''
            }
        }


        stage('Frontend Health Check') {
            steps {

                echo "======================================"
                echo "FRONTEND HEALTH CHECK"
                echo "======================================"

                sh '''
                    echo "Testing frontend..."

                    curl -I -f http://localhost:${FRONTEND_PORT}

                    echo ""

                    echo "Frontend is working successfully"
                '''
            }
        }


        stage('Deployment Information') {
            steps {

                echo "======================================"
                echo "DEPLOYMENT COMPLETED"
                echo "======================================"

                echo "Frontend:"
                echo "http://${EC2_PUBLIC_IP}"

                echo ""

                echo "Backend:"
                echo "http://${EC2_PUBLIC_IP}:${BACKEND_PORT}"

                echo ""

                echo "Swagger:"
                echo "http://${EC2_PUBLIC_IP}:${BACKEND_PORT}/docs"

                echo ""

                echo "Jenkins:"
                echo "http://${EC2_PUBLIC_IP}:8080"

                echo "======================================"
            }
        }
    }


    post {

        success {

            echo """
========================================
        DEPLOYMENT SUCCESSFUL
========================================

Application:
http://${EC2_PUBLIC_IP}

Backend:
http://${EC2_PUBLIC_IP}:${BACKEND_PORT}

Swagger:
http://${EC2_PUBLIC_IP}:${BACKEND_PORT}/docs

========================================
"""
        }


        failure {

            echo """
========================================
        DEPLOYMENT FAILED
========================================

Run these commands on EC2:

cd ${PROJECT_DIR}

docker compose ps

docker compose logs --tail=100

docker logs health-db

docker logs health-backend

docker logs health-frontend

========================================
"""
        }
    }
}
