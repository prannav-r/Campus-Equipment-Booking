pipeline {
agent any

environment {
    IMAGE_NAME = 'campus-equipment-booking:set-e'
}

stages {
    stage('Checkout') {
        steps {
            echo 'Source code is checked out by Jenkins.'
        }
    }

    stage('Verify Java and Maven') {
        steps {
            sh 'java -version'
            sh 'mvn -version'
        }
    }

    stage('Test') {
        steps {
            sh 'mvn clean test'
        }
    }

    stage('Package') {
        steps {
            sh 'mvn package -DskipTests'
        }
    }

    stage('Build Docker Image') {
        steps {
            sh 'docker build -t $IMAGE_NAME .'
        }
    }

    stage('Verify Docker Image') {
        steps {
            sh 'docker images campus-equipment-booking'
        }
    }
}

post {
    success {
        echo 'CI pipeline completed successfully.'
    }

    failure {
        echo 'CI pipeline failed. Check the stage logs.'
    }
}

}
