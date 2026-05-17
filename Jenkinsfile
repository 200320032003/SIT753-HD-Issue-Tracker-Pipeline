pipeline {
    agent any

    environment {
        APP_NAME = 'sit753-issue-tracker'
        IMAGE_NAME = 'sit753-issue-tracker'
        IMAGE_TAG = "1.0-${env.BUILD_NUMBER}"
        TEST_CONTAINER = 'sit753-app-test'
        RELEASE_CONTAINER = 'sit753-app-release'
        MONGO_URI = 'mongodb://host.docker.internal:27017/sit753_issue_tracker'
    }

    stages {
        stage('Build') {
            steps {
                echo 'Building application Docker image...'
                echo "Branch: ${env.BRANCH_NAME}"
                echo "Build number: ${env.BUILD_NUMBER}"
                bat 'docker build -t %IMAGE_NAME%:%IMAGE_TAG% .'
            }
        }

        stage('Test') {
            steps {
                echo 'Running automated CRUD tests...'
                bat 'npm install'
                bat 'npm test'
            }
        }

        stage('Code Quality') {
            steps {
                echo 'Running ESLint code quality analysis...'
                bat 'npm run lint'
            }
        }

        stage('Security') {
            steps {
                echo 'Running npm audit security scan...'
                bat 'npm audit || exit 0'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application to test Docker environment...'
                bat 'docker rm -f %TEST_CONTAINER% || exit 0'
                bat 'docker run -d --name %TEST_CONTAINER% -p 3001:3000 -e MONGO_URI=%MONGO_URI% %IMAGE_NAME%:%IMAGE_TAG%'
            }
        }

        stage('Release') {
            steps {
                echo 'Promoting application image to release version...'
                bat 'docker tag %IMAGE_NAME%:%IMAGE_TAG% %IMAGE_NAME%:release-%BUILD_NUMBER%'
                bat 'docker rm -f %RELEASE_CONTAINER% || exit 0'
                bat 'docker run -d --name %RELEASE_CONTAINER% -p 3002:3000 -e MONGO_URI=%MONGO_URI% %IMAGE_NAME%:release-%BUILD_NUMBER%'
            }
        }

        stage('Monitoring') {
            steps {
                echo 'Checking application health endpoint...'
                bat 'curl http://localhost:3002/health'
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed. Cleaning up test deployment container...'
            bat 'docker rm -f %TEST_CONTAINER% || exit 0'
        }

        success {
            echo 'Pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Check Jenkins logs for details.'
        }
    }
}