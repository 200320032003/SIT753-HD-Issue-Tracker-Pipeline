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
                echo "Build number: ${env.BUILD_NUMBER}"
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Test') {
            steps {
                echo 'Installing dependencies and running automated CRUD tests...'
                sh 'npm install'
                sh 'npm test'
            }
        }

        stage('Code Quality') {
            steps {
                echo 'Running ESLint code quality analysis...'
                sh 'npm run lint'
            }
        }

        stage('Security') {
            steps {
                echo 'Running npm audit security scan...'
                sh 'npm audit || true'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application to test Docker environment...'
                sh 'docker rm -f $TEST_CONTAINER || true'
                sh 'docker run -d --name $TEST_CONTAINER -p 3001:3000 -e MONGO_URI=$MONGO_URI $IMAGE_NAME:$IMAGE_TAG'
            }
        }

        stage('Release') {
            steps {
                echo 'Promoting application image to release version...'
                sh 'docker tag $IMAGE_NAME:$IMAGE_TAG $IMAGE_NAME:release-$BUILD_NUMBER'
                sh 'docker rm -f $RELEASE_CONTAINER || true'
                sh 'docker run -d --name $RELEASE_CONTAINER -p 3002:3000 -e MONGO_URI=$MONGO_URI $IMAGE_NAME:release-$BUILD_NUMBER'
            }
        }

        stage('Monitoring') {
            steps {
                echo 'Checking released application health endpoint...'
                sh 'sleep 5'
                sh 'curl http://host.docker.internal:3002/health'
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed. Cleaning up test deployment container...'
            sh 'docker rm -f $TEST_CONTAINER || true'
        }

        success {
            echo 'Pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Check Jenkins logs for details.'
        }
    }
}